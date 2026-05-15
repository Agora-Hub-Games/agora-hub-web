---
title: Admin accounts
description: Come creare e gestire gli account admin per i brand clienti.
navigation:
  title: Account admin
  icon: i-lucide-users
---

# Admin accounts

Admin users are created in two steps: first in Supabase Auth (so they have login credentials), then linked to a brand in the admin panel.

## Step 1 — Create the user in Supabase Auth

1. Open the Supabase dashboard → **Authentication** → **Users**.
2. Click **Invite user** and enter the client's email address.
3. Supabase sends a magic-link email; the user sets their own password on first login.

Alternatively, for internal accounts you can use **Create user** (manual password).

## Step 2 — Link the user to a brand

1. Log in to the admin panel as superadmin.
2. Go to **Accessi** (top navigation — visible only to superadmin).
3. Enter the user's email address, select their brand from the dropdown, and click **Aggiungi admin**.

This calls the `upsert_brand_admin` DB function which writes a row to `admin_profiles`:

| Column | Value |
|--------|-------|
| `id` | auth user UUID |
| `email` | user's email |
| `brand_id` | selected brand UUID (NULL = superadmin) |

## Account types

| Type | `brand_id` | Access |
|------|-----------|--------|
| Superadmin | NULL | All brands, admin management |
| Brand admin | UUID | Their brand's editor only |

## Branded login URL

Each brand has its own login URL:

```
https://your-admin-domain/#/brand/<brand-slug>
```

Send this URL to the client. When they open it they see a login page scoped to their brand. After login they land directly in their game editor (no dashboard).

The admin panel's **Accessi** page has a copy button next to each admin entry to copy their branded URL quickly.

## Removing access

In **Accessi**, click the trash icon next to an admin to remove their `admin_profiles` row. Their Supabase Auth account remains (it can be deleted from the Supabase dashboard if needed).

## Supabase RLS

Brand-scoped admins can only read/write rows for their own `brand_id`. The policies are defined in `supabase/migrations/003_brand_access.sql`. Superadmins (brand_id IS NULL) bypass brand filtering via a separate policy.
