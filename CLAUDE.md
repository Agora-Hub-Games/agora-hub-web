# agora-hub-web — Landing Page & Docs

## Repo boundary — IMPORTANTE
Questo repo contiene **solo** landing page e documentazione pubblica. Se l'utente chiede qualcosa che riguarda il **widget**, l'**admin panel**, **Supabase**, **Stripe** o il **backend**, rispondi che va fatto nell'altro repo (`agora-hub-admin`) e non toccare nulla qui.

Appartiene a questo repo: landing page marketing, docs pubbliche (Nuxt Content), componenti UI statici.
NON appartiene a questo repo: widget IIFE, admin panel, Supabase migrations, edge functions, billing logic.

## What this is
Landing page commerciale e documentazione pubblica di Agora Games.
Stack: Nuxt 4 + `@nuxt/ui` + `@nuxt/content` v3. Prerendering statico su Cloudflare Pages (o CDN equivalente).

## Repo structure

```
app/
  pages/
    index.vue           — Landing page (hero, features, pricing, CTA)
    docs/
      index.vue         — Indice docs (lista di tutte le guide)
      [...slug].vue     — Singola pagina doc (renders markdown via ContentRenderer)
  components/
    AppLogo.vue         — Logo Agora Games
    TemplateMenu.vue    — Menu di navigazione
  assets/css/main.css   — Stili globali (Tailwind)
  app.config.ts         — Colori @nuxt/ui (primary, neutral)
  app.vue               — Root app
content/
  docs/                 — Guide tecniche in Markdown (Nuxt Content v3)
    integration.md      — Guida all'integrazione widget
    client-onboarding.md — Onboarding nuovo brand cliente
    admin-accounts.md   — Creazione e gestione account admin
    database.md         — Schema DB, migrazioni, operazioni comuni
nuxt.config.ts          — Moduli, content collections, routeRules prerender
```

## Critical constraints

### 1. Content collection
I file markdown in `content/docs/` appartengono alla collection `docs` (definita in `nuxt.config.ts`).
Usa sempre `queryCollection('docs')` nelle pagine, mai `queryContent()` (API v2, deprecata).

### 2. Frontmatter obbligatorio
Ogni file markdown in `content/docs/` deve avere:
```yaml
---
title: ...
description: ...
navigation:
  title: ...   # label breve per menu/indice
  icon: i-lucide-... # icona Lucide
---
```

### 3. Link tra docs
I link interni tra pagine docs usano path Nuxt (`/docs/integration`), NON path relativi del filesystem (`./integration.md`).

### 4. Prerendering
Tutte le route `/docs/**` sono prerendered (definito in `nuxt.config.ts`). Non aggiungere logica server-side o API routes in questo repo: è un sito statico.

### 5. Stile
Usa solo componenti `@nuxt/ui` (`UPage`, `UPageHeader`, `UPageBody`, `UCard`, `UIcon`, ecc.).
Non installare altre librerie UI.

## Repo correlati

| Repo | Contenuto |
|---|---|
| `agora-hub-admin` | Widget IIFE + admin panel Vue 3 + Supabase migrations/functions |
| `agora-hub-web` | Questo repo — landing + docs |

## How this file is maintained

**Quando viene segnalato un errore**, aggiungilo subito alla sezione "Common mistakes" con il formato "non fare X / fai Y".

## Common mistakes to avoid

- Usare `queryContent()` invece di `queryCollection('docs')` — quella è l'API v2, non funziona con Nuxt Content v3
- Aggiungere link interni tra docs con path relativi (`.md`) invece di path Nuxt (`/docs/slug`)
- Dimenticare il frontmatter `title`/`description` in un nuovo file markdown
