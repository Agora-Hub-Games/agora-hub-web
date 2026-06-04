---
title: Database
description: Schema del database Supabase, migrazioni e operazioni comuni.
navigation:
  title: Database
  icon: i-lucide-database
---

# Database management

## Schema overview

```
brands
  id              UUID PK
  slug            TEXT UNIQUE       — URL identifier (e.g. "fashion-demo")
  name            TEXT
  logo_url        TEXT nullable
  primary_color   TEXT              — hex
  secondary_color TEXT
  background_color TEXT
  text_color      TEXT
  active          BOOL
  created_at      TIMESTAMPTZ

brand_games
  id              UUID PK
  brand_id        UUID FK → brands.id
  game_type_id    TEXT              — e.g. "catcher", "runner"
  name            TEXT nullable     — custom display name
  active          BOOL
  config          JSONB             — GameTypeConfig shape
  created_at      TIMESTAMPTZ

game_types
  id              TEXT PK           — "catcher", "runner"
  default_config  JSONB

coupons
  id              UUID PK
  brand_id        UUID FK → brands.id
  game_type_id    TEXT FK → game_types.id
  code            TEXT
  description     TEXT
  coupon_type     TEXT              — "static" | "pool"
  active          BOOL
  max_uses        INT nullable
  uses_count      INT
  created_at      TIMESTAMPTZ

admin_profiles
  id              UUID PK FK → auth.users.id
  email           TEXT
  display_name    TEXT nullable
  brand_id        UUID nullable FK → brands.id   — NULL = superadmin
  created_at      TIMESTAMPTZ

── Billing ──────────────────────────────────────────────────────────────────

billing_plans
  id                  TEXT PK           — "starter", "pro", "enterprise"
  name                TEXT
  monthly_price_cents INT               — e.g. 2900 = €29/month
  included_sessions   INT               — sessions topped up on monthly renewal (0 = none)
  stripe_price_id     TEXT nullable     — Stripe recurring Price ID
  active              BOOL
  created_at          TIMESTAMPTZ

brand_subscriptions
  id                      UUID PK
  brand_id                UUID UNIQUE FK → brands.id
  plan_id                 TEXT nullable FK → billing_plans.id
  status                  TEXT          — "active" | "trialing" | "past_due" | "canceled" | "inactive"
  stripe_customer_id      TEXT nullable
  stripe_subscription_id  TEXT nullable UNIQUE
  current_period_start    TIMESTAMPTZ nullable
  current_period_end      TIMESTAMPTZ nullable
  canceled_at             TIMESTAMPTZ nullable
  created_at              TIMESTAMPTZ
  updated_at              TIMESTAMPTZ

visit_packages
  id              TEXT PK           — "pack_1k", "pack_5k", "pack_20k"
  name            TEXT
  sessions_count  INT
  price_cents     INT               — e.g. 1900 = €19
  stripe_price_id TEXT nullable     — Stripe one-time Price ID
  active          BOOL
  created_at      TIMESTAMPTZ

brand_visit_credits
  brand_id            UUID PK FK → brands.id
  sessions_remaining  INT           — current balance (never negative below 0)
  sessions_total      INT           — lifetime credits purchased/added
  updated_at          TIMESTAMPTZ

billing_events
  id              UUID PK
  brand_id        UUID nullable FK → brands.id
  event_type      TEXT              — see event types below
  stripe_event_id TEXT nullable UNIQUE  — Stripe event ID for idempotency
  amount_cents    INT nullable
  sessions_delta  INT nullable      — positive = added, negative = consumed
  metadata        JSONB
  created_at      TIMESTAMPTZ

billing_events.event_type values:
  subscription_created, subscription_canceled, subscription_renewed
  customer.subscription.created, customer.subscription.updated, customer.subscription.deleted
  pack_purchased, session_consumed, payment_failed

── Newsletter ────────────────────────────────────────────────────────────────

brand_klaviyo_config
  brand_id    UUID PK FK → brands.id
  api_key     TEXT              — Klaviyo private API key (never exposed to widget)
  updated_at  TIMESTAMPTZ

newsletter_signups
  id               UUID PK
  brand_id         UUID nullable FK → brands.id
  game_session_id  UUID nullable FK → game_sessions.id
  email            TEXT
  klaviyo_synced   BOOL          — true if Klaviyo API call succeeded
  created_at       TIMESTAMPTZ

── Preview & anti-abuse ────────────────────────────────────────────────────────

game_preview_links               — shareable, expiring no-login game previews
  token         TEXT PK          — random; used in the /p/:token public URL
  brand_id      UUID FK → brands.id
  game_id       UUID FK → brand_games.id
  expires_at    TIMESTAMPTZ nullable  — null = never expires
  created_at    TIMESTAMPTZ
  — RLS: brand members manage their own; public reads go through the
    `preview-config` Edge Function (service role), billing bypassed.

widget_ip_plays                  — per-IP play counter for the anti-abuse limit
  id            BIGINT PK (identity)
  brand_id      UUID FK → brands.id
  game_id       UUID FK → brand_games.id
  ip_hash       TEXT             — salted SHA-256 of the visitor IP (never raw)
  played_at     TIMESTAMPTZ
  — RLS enabled, no policies: only the service role (Edge Function) touches it.
```

## Migrations

Migrations live in `supabase/migrations/` in the `agora-hub-admin` repo. Run them in order via the Supabase dashboard (SQL editor) or CLI:

```bash
supabase db push
```

| File | What it does |
|------|--------------|
| `001_initial.sql` | Core tables: brands, brand_games, coupons, game_types, session_events |
| `002_storage_and_admin.sql` | Storage bucket `brand-assets` + upload policies; admin_profiles table |
| `003_brand_access.sql` | Adds `brand_id` + `display_name` to `admin_profiles`; RLS for brand-scoped admins |
| `004_admin_management.sql` | DB functions: `get_user_id_by_email`, `upsert_brand_admin`, `remove_brand_admin` |
| `005_coupon_per_game.sql` | Adds `game_type_id` to coupons; one coupon per game type per brand |
| `006_brand_assets_bucket.sql` | Extends brand-assets storage policies |
| `007_admin_signup_trigger.sql` | Trigger that auto-creates admin_profile on auth.users insert via invite |
| `008_game_name_and_instances.sql` | Adds `name` column to brand_games for custom display labels |
| `009_billing.sql` | Billing system: billing_plans, brand_subscriptions, visit_packages, brand_visit_credits, billing_events; RLS policies; seed data for plans and packages |
| `010_billing_rpc.sql` | Atomic RPC functions: `check_and_consume_brand_session` (widget gate + credit debit, callable by anon), `increment_brand_credits` (webhook top-up) |
| `011_newsletter.sql` | Newsletter integration: `brand_klaviyo_config` (per-brand Klaviyo API key, server-side only), `newsletter_signups` (audit log), RLS policies |
| `037_analytics_bounce_rate.sql` | Analytics RPC `get_brand_bounce_rate(p_brand_id)` → `ready_sessions` / `started_sessions` from `session_events` (game_ready vs game_start by `properties->>'brand_id'`, excludes editor placement); powers the widget bounce-rate metric |
| `038_game_preview_links.sql` | `game_preview_links` table (shareable, expiring no-login preview links) + RLS scoped to brand members. Public resolution via the `preview-config` Edge Function (E-06) |
| `039_ip_play_limit.sql` | `widget_ip_plays` table + atomic RPC `check_and_record_ip_play(p_brand_id, p_game_id, p_ip_hash, p_max, p_window_hours)` for the per-IP play limit; EXECUTE restricted to the service role. Enforced by the `check-ip-limit` Edge Function (E-24) |

## Config JSONB structure (`brand_games.config`)

```json
{
  "gameplay": {
    "winThreshold": 10,
    "gameDuration": 30,
    "maxMisses": 5,
    "items": ["🎁", "⭐"],
    "gameMode": "timed",
    "maxPlaysPerPeriod": null,
    "playPeriodHours": 24,
    "maxPlaysPerIp": null
  },
  "content": {
    "title": "...",
    "subtitle": "...",
    "startLabel": "Inizia a giocare",
    "wonTitle": "Hai vinto!",
    "wonSubtitle": "Ecco il tuo premio:",
    "ctaLabel": "Scopri di più",
    "ctaUrl": "https://...",
    "lostTitle": "Quasi!",
    "lostSubtitle": "Riprova, ci sei quasi!",
    "retryLabel": "Riprova",
    "instructionsMove": "...",
    "instructionsKeys": "...",
    "prizeLabel": "Premio in palio"
  },
  "audio": { "enabled": false, "musicVolume": 0.4 },
  "appearance": {
    "backgroundType": "gradient",
    "backgroundValue": "linear-gradient(...)",
    "basketColor": "#E94560"
  },
  "runnerConfig": {
    "gravity": 0.75,
    "jumpForce": 13.5,
    "hasSlide": true,
    "hasDoubleJump": false,
    "obstacleImageUrls": ["https://..."],
    "obstacleWeights": [3]
  },
  "schedule": {
    "activeFrom": "2026-11-29T09:00",
    "activeUntil": "2026-12-02T23:59",
    "weekly": { "mon": [9, 22], "sat": null }
  },
  "targeting": {
    "minCartValue": 50,
    "utmSources": ["newsletter", "instagram"]
  }
}
```

**Optional config blocks** (editor features, all stored in `brand_games.config`):

| Block | Purpose |
|-------|---------|
| `schedule` | Activation window (`activeFrom`/`activeUntil`) + `weekly` per-day active hours (`[startHour, endHour]` or `null` = closed). Enforced widget-side before the billing gate. |
| `targeting` | Show the widget only when the host cart total ≥ `minCartValue` and/or `utm_source` ∈ `utmSources`. Evaluated in the widget entry. |
| `gameplay.maxPlaysPerIp` | Per-IP play cap (anti-abuse), enforced via the `check-ip-limit` Edge Function over `playPeriodHours`. |
| `runnerConfig.obstacleWeights` | Spawn weight (1–5) per obstacle image, index-aligned with `obstacleImageUrls`; missing = 3. |
| `coupons` | Tier-based rewards (`[{ threshold, code, description }]`). |
| `configB` | A/B variant B overrides (partial `GameTypeConfig`). |

## Common operations in Supabase dashboard

**Add a new brand:**
```sql
INSERT INTO brands (id, slug, name, primary_color, secondary_color, background_color, text_color, active)
VALUES (gen_random_uuid(), 'my-brand', 'My Brand', '#6C3CE1', '#F5A623', '#1A0533', '#FFFFFF', true);
```

**Set Stripe Price IDs on plans (required before going live):**
```sql
UPDATE billing_plans SET stripe_price_id = 'price_xxx' WHERE id = 'starter';
UPDATE billing_plans SET stripe_price_id = 'price_yyy' WHERE id = 'pro';
UPDATE billing_plans SET stripe_price_id = 'price_zzz' WHERE id = 'enterprise';

UPDATE visit_packages SET stripe_price_id = 'price_aaa' WHERE id = 'pack_1k';
UPDATE visit_packages SET stripe_price_id = 'price_bbb' WHERE id = 'pack_5k';
UPDATE visit_packages SET stripe_price_id = 'price_ccc' WHERE id = 'pack_20k';
```

**Manually grant trial access to a brand:**
```sql
INSERT INTO brand_subscriptions (brand_id, status)
VALUES ('<brand_uuid>', 'trialing')
ON CONFLICT (brand_id) DO UPDATE SET status = 'trialing';
```

**Check billing status for a brand:**
```sql
SELECT
  b.slug,
  bs.status,
  bp.name AS plan,
  bvc.sessions_remaining
FROM brands b
LEFT JOIN brand_subscriptions bs ON bs.brand_id = b.id
LEFT JOIN billing_plans bp       ON bp.id = bs.plan_id
LEFT JOIN brand_visit_credits bvc ON bvc.brand_id = b.id
WHERE b.slug = 'my-brand';
```

**Check which admins have access to a brand:**
```sql
SELECT ap.email, ap.display_name, b.name AS brand
FROM admin_profiles ap
LEFT JOIN brands b ON b.id = ap.brand_id;
```
