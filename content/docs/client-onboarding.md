---
title: Client onboarding
description: Checklist completa per aggiungere un nuovo brand cliente alla piattaforma.
navigation:
  title: Onboarding
  icon: i-lucide-list-checks
---

# Client onboarding

Complete checklist for adding a new brand client to the platform.

> **Note**: Steps 1–4 currently require superadmin SQL access. Self-service signup is planned (see Notion: 🚀 Self-service brand signup).

## 1. Create the brand in Supabase

Open the Supabase SQL editor and run:

```sql
INSERT INTO brands (id, slug, name, primary_color, secondary_color, background_color, text_color, active)
VALUES (
  gen_random_uuid(),
  'brand-slug',       -- URL-safe, lowercase, no spaces
  'Brand Name',
  '#6C3CE1',          -- primary color
  '#F5A623',          -- secondary color (also default basket color)
  '#1A0533',          -- widget background
  '#FFFFFF',
  true
);
```

## 2. Add at least one game

```sql
INSERT INTO brand_games (id, brand_id, game_type_id, active, config)
VALUES (
  gen_random_uuid(),
  (SELECT id FROM brands WHERE slug = 'brand-slug'),
  'catcher',
  true,
  '{}'::jsonb        -- empty = uses CATCHER_DEFAULTS
);
```

## 3. Add a coupon

```sql
INSERT INTO coupons (id, brand_id, code, description, coupon_type, active)
VALUES (
  gen_random_uuid(),
  (SELECT id FROM brands WHERE slug = 'brand-slug'),
  'WELCOME10',
  '10% di sconto sul primo ordine',
  'static',
  true
);
```

### Pool coupons (codici univoci per ogni vincitore)

Se il brand vuole coupon diversi per ogni vincitore, usa `coupon_type = 'pool'`.
L'UI per gestire il pool di codici è in roadmap. Per ora, caricare i codici tramite SQL:

```sql
-- Prima crea il coupon pool
INSERT INTO coupons (id, brand_id, code, description, coupon_type, active)
VALUES (gen_random_uuid(), (SELECT id FROM brands WHERE slug = 'brand-slug'),
        '', '10% coupon esclusivo', 'pool', true);

-- Poi aggiungi i codici alla tabella pool (quando implementata)
-- Attualmente il tipo 'pool' non ha ancora un pool di codici: usa 'static' in produzione.
```

## 4. Create the admin account

See [admin-accounts](/docs/admin-accounts) for full instructions.

Short version:
1. Go to Supabase → Authentication → Users → Invite user (enter client email).
2. Open the admin panel at `/#/admins`, add the user email and select the brand.
3. Send the client their branded login URL: `https://your-admin-domain/#/brand/brand-slug`.

## 5. Configure the brand via admin panel

Log in as superadmin and go to the brand's editor:

- **Aspetto** — upload logo, set brand colors, background, optional basket image and font.
- **Testi** — customize all screen copy: title, start button, won/lost messages.
- **Gameplay** — tune difficulty (speed, lives, duration).
- **Audio** — upload optional background music and sound effects.
- **Coupon** — verify the coupon code and description are correct.
- **Newsletter** — configure Klaviyo list ID and whether email capture is required or optional.

Click **Salva** after each section.

## 6. Export the embed snippet

In the editor header click **`</> Integra`**. Copy the snippet and send it to the client's developer:

```html
<div id="agora-games-widget"></div>
<script
  src="https://cdn.agoragames.io/v1/widget.js"
  data-brand="brand-slug"
  data-placement="homepage"
  data-container="#agora-games-widget"
></script>
```

The developer drops this into any page where the widget should appear.

See [integration](/docs/integration) for framework-specific examples (Shopify, WooCommerce, Next.js).

## 7. Verifica in produzione

Apri il widget sul sito del cliente e conferma:
- Brand colors and logo appear correctly.
- Game loads and plays through to the won/lost screen.
- Coupon code is displayed on the won screen.
- CTA button links to the correct URL.
- (If newsletter enabled) email capture step appears before coupon reveal.
- Tracking events are firing in GA4 / GTM (check Network tab or console).

## 8. Billing e crediti visite

Il widget si carica solo se il brand ha un abbonamento attivo **oppure** crediti visite disponibili. Senza nessuno dei due, il widget non si avvia.

### Modelli di accesso

| Modello | Come funziona |
|---|---|
| **Abbonamento mensile** | Pagamento ricorrente Stripe. Ogni rinnovo mensile accredita automaticamente le sessioni incluse nel piano (es. 5.000 per il piano Pro). |
| **Pacchetti visite** | Acquisto una-tantum di un blocco di sessioni (es. 1.000, 5.000, 20.000). Le sessioni vengono scalate a ogni caricamento del widget. |

I due modelli sono compatibili: un brand può avere sia un abbonamento attivo sia crediti aggiuntivi acquistati come pacchetti.

### Attivare il billing per un nuovo brand

1. **Crea i Price objects in Stripe** — accedi al dashboard Stripe e crea:
   - 3 prezzi ricorrenti mensili (uno per ogni piano: Starter, Pro, Enterprise)
   - 3 prezzi one-time (uno per ogni pacchetto visite: 1k, 5k, 20k)
2. **Aggiorna i Price IDs in Supabase** — nel pannello admin, sezione **Billing** → zona superadmin in fondo alla pagina, incolla i `stripe_price_id` corrispondenti su ogni piano e pacchetto.
3. **Il brand acquista il piano** — da `/#/billing`, seleziona il brand e clicca "Attiva piano" oppure "Acquista pacchetto". Stripe Checkout gestisce il pagamento; al ritorno i crediti vengono accreditati automaticamente.

### Trial gratuito per un nuovo brand (manuale)

```sql
INSERT INTO brand_subscriptions (brand_id, status)
VALUES ('<brand_uuid>', 'trialing')
ON CONFLICT (brand_id) DO UPDATE SET status = 'trialing';
```

> Questo passaggio verrà automatizzato con il self-service signup (Q2 2026).

### Cosa succede quando i crediti si esauriscono

- Il banner **"Crediti in esaurimento"** appare in `/#/billing` quando i crediti scendono sotto il 10% del totale acquistato.
- Quando i crediti arrivano a zero (e non c'è un abbonamento attivo), il widget smette di caricarsi e mostra un errore generico all'utente finale.
- Per ricaricare: acquista un nuovo pacchetto visite da `/#/billing` o rinnova l'abbonamento.

## 9. Newsletter & Klaviyo

Il widget supporta la raccolta email opzionale prima del reveal del coupon (schermata "Won").

### Setup lato Agora (superadmin)

1. In `/#/editor/:brand/:gameId`, sezione **Newsletter**:
   - Abilita la raccolta email
   - Imposta se è obbligatoria o facoltativa (il giocatore può saltarla)
   - Inserisci il **Klaviyo List ID** del brand
2. In **Impostazioni Brand** (o tramite SQL), salva la **Klaviyo Private API Key** del brand in `brand_klaviyo_config`. Questa chiave viene usata solo lato server (Edge Function) e non è mai esposta al widget.

### Setup lato cliente (brand admin)

Il brand admin trova la sezione Newsletter nell'editor. Deve fornire:
- La Klaviyo Private API Key (da Klaviyo → Account → API Keys)
- Il List ID della lista a cui aggiungere i contatti

### Flusso tecnico

Quando un giocatore inserisce l'email:
1. Il widget chiama `POST /functions/v1/subscribe-newsletter`
2. L'Edge Function legge la Klaviyo API key da `brand_klaviyo_config` (mai esposta al client)
3. Aggiunge il contatto alla lista Klaviyo specificata
4. Salva il signup in `newsletter_signups` con `klaviyo_synced = true/false`

### Verifica

Dopo il test, controlla in Klaviyo che il contatto sia apparso nella lista corretta.

## 10. Tracking & Analytics

Il widget emette eventi standard compatibili con GA4, Segment, GTM. Per attivare il tracking personalizzato:

```html
<script>
  window.AgoraGamesTracking = {
    onEvent: function(event) {
      // GA4
      gtag('event', event.name, event.properties)
      // oppure GTM
      window.dataLayer.push({ event: event.name, ...event.properties })
    }
  }
</script>
```

Per la lista completa degli eventi, vedi [integration](/docs/integration#tracking-personalizzato).

Gli eventi `coupon_redeemed` e `post_game_conversion` richiedono implementazione lato sito cliente (fire quando il coupon viene usato in checkout).
