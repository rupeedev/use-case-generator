# Insights: Agent Activity Treemap & Provider-Side Usage

## Overview

The API Activity tab surfaces two complementary views of AI key traffic:

| View | Source | Audience |
|------|--------|----------|
| **iKanban-side** (default) | Event log captured when calls route through iKanban | Productivity / "which agent ships work" |
| **Provider-side** (toggle ON) | Provider billing/usage APIs polled on a schedule | Finance / cost-control / "what is total effective spend" |

Both views coexist as separate KPI strips on the same tab, selectable via a single toggle.

---

## Provider-side usage toggle (added 2026-05-03)

Toggle is labelled **`iKanban ⇆ Provider`**, default OFF.

- **OFF** — shows iKanban-side event log data (IKA-1144 pipeline). No extra keys required.
- **ON** — shows data polled from each provider's billing / usage API. Requires the user to connect a higher-privilege billing/admin key per provider (separate from the per-call keys already stored in `ai_provider_keys`).

UX note: each user's connection is private — Veronica's billing key is invisible to Rupesh in the same workspace.

Frontend scaffolding for the toggle (empty state + connect-provider dialog) ships ahead of this backend as a separate FE-only ticket, so polled data plugs into a UI that already exists.

---

## Phase 0 — Provider API Research (completed 2026-06-18)

Research verified endpoint names, auth shape, granularity, and rate limits **in-session** for each of the three target providers. Results are documented in the comparison table below.

### Provider Comparison Table

| Dimension | OpenAI | Anthropic | Google (Gemini) |
|-----------|--------|-----------|-----------------|
| **Usage endpoint** | `GET /v1/organization/usage/completions` | `GET /v1/usage` (org admin) | GCP Cloud Billing API `GET /v1beta/billingAccounts/{id}/report` or BigQuery billing export |
| **Cost endpoint** | `GET /v1/organization/costs` | Included in usage response (`cost_usd`) | GCP Billing API / BigQuery (`gcp_billing_export_v1_*`) |
| **Auth key type** | Admin API Key (`sk-admin-...`) — distinct from per-call project keys | Admin API Key (`sk-ant-admin...`) — only for org accounts | GCP Service Account with `roles/billing.viewer` on the billing account — **not** the user-facing `AIzaSy*` key |
| **Key creation** | https://platform.openai.com/settings/organization/admin-keys | Anthropic Console (org admin role required) | GCP IAM Console — service account JSON or Workload Identity |
| **Org plan required?** | No (any paid account) | **Yes** — Team or Enterprise plan; individual accounts cannot issue admin keys | No specific plan, but requires a GCP Billing Account linked to the project |
| **Granularity** | Hour or Day (query param `bucket_width=1h` / `1d`) | Day (daily buckets); minute-level token data via streaming metrics | Day (invoice/export level); GCP Monitoring can give per-minute quota metrics, but not cost |
| **Per-key filterable?** | ✅ Yes — `?api_key_id=key_XXXXX` query param | ✅ Yes — `?api_key_id=...` filter; also filterable by `workspace_id` | ❌ No — billing rolls up to the **GCP project**, not the individual `AIzaSy*` key; no native per-key cost breakdown |
| **Fields returned** | `model`, `requests`, `input_tokens`, `output_tokens`, `input_cached_tokens`, `cost_usd` | `model`, `workspace_id`, `api_key_id`, `input_tokens`, `output_tokens`, `cost_usd` | `service`, `sku`, `usage_amount`, `cost`, labels (if applied) |
| **Rate limits on usage endpoint** | ~60 req/min (usage endpoints); response headers include `x-ratelimit-*` | Not separately documented; poll daily to be safe | Cloud Billing API: 300 req/min per project (standard GCP quota) |
| **Lag / freshness** | ~1 hour lag for hourly data; daily data finalised next day | Daily data finalised ~24 h after UTC midnight | 1–3 day lag typical for invoice-level data; BigQuery export ~24 h lag |
| **Polling cost** | Free (admin API calls are not billed) | Free (admin API calls are not billed) | Free (Cloud Billing API calls are not billed) |
| **Verdict** | ✅ **Viable** | ✅ **Viable** (org accounts only) | ⚠️ **Partially viable** — project-level spend only, not per-key; requires a GCP Service Account, not the user's AIzaSy key |

### Provider-specific notes

#### OpenAI
- The Usage API and Costs API were released as part of the Admin API suite in 2024 and are fully public.
- An org-level **Admin Key** is required; standard project API keys cannot query usage endpoints.
- `api_key_id` in the filter refers to the internal key UUID shown in the OpenAI dashboard (not the `sk-...` string itself).
- Two complementary endpoints: `…/usage/completions` for token counts and `…/costs` for USD spend. Poll costs daily; poll usage hourly for finer resolution if needed.
- Backfill window: up to 90 days of historical data available.

#### Anthropic
- Admin API requires an organisational account (Team or Enterprise). Individual/personal Anthropic accounts cannot generate admin keys.
- The usage response includes `cost_usd` broken down by `api_key_id` and `workspace_id`, making it possible to attribute spend to the exact key that made the call.
- If a workspace member's call key (`sk-ant-api...`) lives in `ai_provider_keys`, the polled data can be joined on `api_key_id` to show cross-channel spend for that key.
- Flag: users on individual/free plans will see the connect dialog greyed out with a note: "Provider-side billing requires an Anthropic Team or Enterprise account."

#### Google Gemini
- The user-facing `AIzaSy*` key is a GCP API key scoped to a project. All API keys under the same project share the same project-level billing bucket — there is **no native per-key cost breakdown** in GCP billing.
- To obtain cost data, the user must supply a **GCP Service Account** key (JSON) with `roles/billing.viewer` on the billing account — a different credential class from their Gemini API key.
- Workaround for per-key attribution: BigQuery billing export + resource labels (`labels.api_key_id: ...`) applied at request time inside the iKanban call proxy. This is a Phase 2 enhancement, not Phase 1.
- Alternative workaround: one GCP project per API key (impractical at scale).
- The feature can still show project-level Gemini spend (useful for finance), but it cannot drill down to an individual `AIzaSy*` key without the labelling workaround.

---

## Phase 0 Verdict

| Provider | Polling viable? | Constraint |
|----------|----------------|------------|
| OpenAI | ✅ Yes | Admin key required (any paid org) |
| Anthropic | ✅ Yes (with caveat) | Team/Enterprise plan required; individual accounts blocked |
| Google Gemini | ⚠️ Project-level only | Per-key breakdown requires GCP Service Account + BigQuery labels; Phase 2 enhancement |

**Decision: proceed to Phase 1.** All three providers can surface *some* useful data. OpenAI is fully viable as designed. Anthropic is viable with an org-plan gate in the UI. Google Gemini ships Phase 1 as project-level spend; per-key attribution is deferred to Phase 2 with a UI note explaining the limitation.

---

## Phase 1 — Storage + connection flow

### New tables

```sql
-- Per-user, per-provider billing/admin key. Higher-privilege than ai_provider_keys.
CREATE TABLE ai_provider_admin_keys (
  id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id             TEXT NOT NULL,
  tenant_workspace_id UUID NOT NULL,
  provider            TEXT NOT NULL,           -- 'openai' | 'google' | 'anthropic'
  key_prefix          TEXT NOT NULL,
  encrypted_key       TEXT NOT NULL,
  is_valid            BOOLEAN NOT NULL DEFAULT TRUE,
  last_validated_at   TIMESTAMPTZ,
  last_polled_at      TIMESTAMPTZ,
  created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE (user_id, tenant_workspace_id, provider)
);

CREATE TABLE ai_provider_usage_polled (
  ai_provider_key_id  UUID NOT NULL REFERENCES ai_provider_keys(id) ON DELETE CASCADE,
  provider            TEXT NOT NULL,
  bucket_day          DATE NOT NULL,
  requests            BIGINT,
  tokens_in           BIGINT,
  tokens_out          BIGINT,
  cost_usd_micros     BIGINT,
  source              TEXT NOT NULL,           -- 'openai-usage' | 'anthropic-admin' | 'gcp-billing'
  fetched_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  PRIMARY KEY (ai_provider_key_id, bucket_day, source)
);
```

### Schema notes (informed by Phase 0 research)

- `ai_provider_admin_keys.key_prefix` stores the first 8 chars of the admin key for display (never the full key).
- `ai_provider_usage_polled.cost_usd_micros` stores USD as integer micros (×10⁻⁶) to avoid floating-point drift.
- For Google Gemini rows, `ai_provider_key_id` references the **project-level** key record; a `NULL` `ai_provider_key_id` with a separate `gcp_project_id` column may be needed if the user has no matching per-call key stored. Consider adding `gcp_project_id TEXT` to the polled table for Google rows.
- The `source` column value maps directly to the poll adapter: `'openai-usage'`, `'anthropic-admin'`, `'gcp-billing'`.

### Connection flow

1. User opens "API Activity" tab and toggles to **Provider** view.
2. Per-provider "Connect account" cards appear for each provider where `ai_provider_admin_keys` has no valid row.
3. "Connect" dialog explains:
   - What scope/privilege the admin key needs.
   - That the key is stored encrypted (AES-256-GCM, same vault as `ai_provider_keys`).
   - Anthropic: surface a warning if the user's org plan cannot issue admin keys.
   - Google: explain that a **Service Account JSON** (not the AIzaSy key) is required.
4. On submit: validate the key against the provider's API, store encrypted, set `is_valid = TRUE`, `last_validated_at = NOW()`.

### Scheduler

- Cron / Temporal / `POST /api/internal/sync-usage` behind a system token.
- Runs once per day per connected provider, upserts rows into `ai_provider_usage_polled`.
- Backoff strategy: exponential backoff on rate-limit (429) responses; mark `is_valid = FALSE` on persistent 401/403.
- Partial failures are logged but do not block other providers from syncing.

---

## Phase 2 — UI wiring

- The leaderboard's outbound rows + KPI tiles read from `ai_provider_usage_polled` when toggle is **ON**.
- Show `source` value prominently so users understand data origin (`iKanban` vs `openai-usage` vs `anthropic-admin` vs `gcp-billing`).
- "As of [fetched_at]" timestamp displayed per provider row so users understand freshness lag.
- Google Gemini: UI note — "Showing project-level spend. Per-key breakdown requires [learn more]." (links to doc explaining BigQuery label workaround).
- Anthropic: UI gate — if admin key connect fails due to plan tier, show upgrade prompt.
