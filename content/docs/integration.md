---
title: Guida all'integrazione
description: Come integrare il widget Agora Games nel tuo e-commerce in meno di 5 minuti.
navigation:
  title: Integrazione
  icon: i-lucide-code-2
---

# Agora Games — Guida all'Integrazione

Questa guida mostra al cliente come integrare il widget di gioco nel proprio e-commerce in meno di 5 minuti.

---

## Integrazione rapida (consigliata)

Incolla questo codice nel tuo sito, dove vuoi che appaia il gioco:

```html
<!-- 1. Posiziona il contenitore dove vuoi il gioco -->
<div id="agora-games"></div>

<!-- 2. Aggiungi il widget prima di </body> -->
<script
  src="https://cdn.agoragames.io/v1/widget.js"
  data-brand="IL_TUO_BRAND_SLUG"
  data-game="catcher"
  data-container="#agora-games"
  async
></script>
```

Sostituisci `IL_TUO_BRAND_SLUG` con lo slug del tuo brand (ti viene fornito da Agora Games).

---

## Opzioni disponibili (attributi `data-*`)

| Attributo | Obbligatorio | Descrizione |
|---|---|---|
| `data-brand` | ✅ | Slug del tuo brand (es. `fashion-demo`) |
| `data-game` | ✅ | Tipo di gioco (es. `catcher`) |
| `data-container` | ❌ | Selettore CSS del contenitore (default: crea un div automaticamente) |
| `data-placement` | ❌ | Etichetta per il tracking (es. `homepage`, `product-page`, `cart`) |

---

## Integrazione avanzata (JavaScript API)

Per casi d'uso più complessi (es. mounting condizionale, SPA):

```html
<div id="my-game-container"></div>

<script src="https://cdn.agoragames.io/v1/widget.js"></script>
<script>
  // Il widget espone AgoraGames.init()
  AgoraGames.init({
    brandSlug: 'IL_TUO_BRAND_SLUG',
    gameType: 'catcher',
    container: '#my-game-container',
    placement: 'homepage-hero',
  })
</script>
```

---

## Tracking personalizzato

Il widget invia eventi a qualsiasi sistema di analytics. Definisci il callback **prima** del tag `<script>` del widget:

```html
<script>
  window.AgoraGamesTracking = {
    onEvent: function(event) {
      // event.name       → nome dell'evento (es. 'game_start', 'coupon_copied')
      // event.properties → oggetto con tutte le proprietà dell'evento

      // Esempio con Google Analytics 4:
      gtag('event', event.name, event.properties)

      // Esempio con Segment:
      analytics.track(event.name, event.properties)

      // Esempio con GTM DataLayer:
      window.dataLayer = window.dataLayer || []
      window.dataLayer.push({ event: event.name, ...event.properties })
    }
  }
</script>
<script src="https://cdn.agoragames.io/v1/widget.js" data-brand="..." data-game="catcher"></script>
```

### Lista completa degli eventi

| Evento | Quando viene inviato |
|---|---|
| `game_loaded` | Il widget è stato caricato nella pagina |
| `game_ready` | Il widget è pronto all'interazione |
| `game_start` | L'utente ha premuto "Inizia a giocare" |
| `game_complete` | La sessione di gioco è terminata |
| `game_exit` | L'utente ha chiuso la scheda durante il gioco |
| `retry_click` | L'utente ha premuto "Riprova" |
| `cta_click` | Click sul pulsante CTA principale |
| `reward_assigned` | Il coupon è stato assegnato al vincitore |
| `coupon_viewed` | Il coupon è stato visualizzato |
| `coupon_copied` | L'utente ha copiato il codice coupon |
| `coupon_redeemed` | Il coupon è stato usato al checkout (fire lato sito cliente) |
| `post_game_conversion` | L'utente ha completato un acquisto dopo il gioco (fire lato sito cliente) |

Ogni evento include queste proprietà standard:

```json
{
  "game_id": "catcher-v1",
  "game_type": "catcher",
  "brand_id": "uuid-del-brand",
  "brand_slug": "fashion-demo",
  "placement": "homepage",
  "session_token": "uuid-sessione",
  "device_type": "mobile",
  "timestamp": 1706000000000
}
```

---

## Personalizzazione del look & feel

Tutte le personalizzazioni grafiche vengono configurate da Agora Games tramite dashboard (o direttamente su Supabase). Il cliente non deve modificare il codice del widget.

Le proprietà personalizzabili per brand:

- **Colori**: primario, secondario, sfondo, testo
- **Logo**: URL dell'immagine del brand
- **Items del gioco**: emoji o URL di immagini
- **Testi**: titolo, sottotitolo, label del CTA
- **Link CTA**: URL a cui mandare l'utente dopo la vittoria
- **Soglia di vittoria**: quanti prodotti catturare per vincere
- **Durata**: secondi del gioco
- **Vite**: quanti errori sono consentiti
- **Coupon**: codice e descrizione dello sconto

---

## Esempi pratici

### Shopify (tema Liquid)

Aggiungi il seguente snippet in `theme.liquid` o in una sezione custom:

```liquid
{% if template == 'index' %}
<div id="agora-games" style="display:flex;justify-content:center;margin:40px 0;"></div>
<script
  src="https://cdn.agoragames.io/v1/widget.js"
  data-brand="{{ shop.metafields.agora.brand_slug }}"
  data-game="catcher"
  data-container="#agora-games"
  data-placement="homepage"
  async
></script>
{% endif %}
```

### WooCommerce (PHP)

```php
// In functions.php
add_action('woocommerce_before_main_content', function() {
    if (is_front_page()) {
        echo '<div id="agora-games"></div>';
        echo '<script src="https://cdn.agoragames.io/v1/widget.js"
              data-brand="IL_TUO_BRAND_SLUG"
              data-game="catcher"
              data-container="#agora-games"
              data-placement="homepage"
              async></script>';
    }
});
```

### Next.js / React

```tsx
'use client'
import { useEffect } from 'react'

export default function AgoraWidget() {
  useEffect(() => {
    window.AgoraGames?.init({
      brandSlug: 'IL_TUO_BRAND_SLUG',
      gameType: 'catcher',
      container: '#agora-games',
      placement: 'homepage',
    })
  }, [])

  return <div id="agora-games" />
}
```

---

## FAQ

**Come firo `coupon_redeemed` e `post_game_conversion`?**
Questi due eventi devono essere inviati dal sito cliente, non dal widget (perché avvengono nel checkout). Esempio:

```js
// Nel tuo checkout, dopo che il coupon è stato applicato:
window.AgoraGamesTracking?.onEvent?.({
  name: 'coupon_redeemed',
  properties: { coupon_code: 'WELCOME10', order_id: '12345' }
})

// Dopo il completamento dell'ordine, se il cliente aveva giocato:
window.AgoraGamesTracking?.onEvent?.({
  name: 'post_game_conversion',
  properties: { order_value: 89.90, currency: 'EUR' }
})
```

**Il widget è GDPR compliant?**
Sì. Non tracciamo dati personali. La session_token è un UUID anonimo generato lato client ad ogni sessione. Se abiliti la newsletter, il widget mostra il form email solo con consenso esplicito dell'utente.

**Il widget funziona su mobile?**
Sì, è mobile-first. Supporta touch drag, mouse move e tastiera.

**Quanto pesa il widget?**
~50KB gzip — si carica in <1s su connessione 3G.

**Posso avere più giochi sulla stessa pagina?**
Sì, chiama `AgoraGames.init()` più volte con container diversi.

**Come aggiorno il coupon o i testi?**
Dalla dashboard Agora Games (o su Supabase). Le modifiche sono live immediatamente senza toccare il codice del sito.
