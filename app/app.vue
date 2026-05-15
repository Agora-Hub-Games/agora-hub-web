<script setup>
useHead({
  meta: [
    { name: 'viewport', content: 'width=device-width, initial-scale=1' }
  ],
  link: [
    { rel: 'icon', href: '/favicon.ico' }
  ],
  htmlAttrs: {
    lang: 'it'
  }
})

const title = 'Agora Games — Gamification widget per il tuo sito'
const description = 'Aggiungi un widget di gamification al tuo sito in 5 minuti. I tuoi clienti giocano, vincono coupon e tornano ad acquistare. Zero sviluppo richiesto.'

useSeoMeta({
  title,
  description,
  ogTitle: title,
  ogDescription: description,
  twitterCard: 'summary_large_image'
})
</script>

<template>
  <UApp>
    <UHeader>
      <template #left>
        <NuxtLink to="/" aria-label="Agora Games">
          <AppLogo class="h-7 w-auto shrink-0" />
        </NuxtLink>
      </template>

      <template #body>
        <UNavigationMenu
          :items="[
            { label: 'Docs', to: '/docs' },
            { label: 'Pricing', to: '#pricing' }
          ]"
          variant="link"
        />
      </template>

      <template #right>
        <UButton
          to="https://admin.agoragames.io"
          target="_blank"
          label="Login"
          color="neutral"
          variant="ghost"
          size="sm"
        />
        <UButton
          to="https://admin.agoragames.io/signup"
          target="_blank"
          label="Richiedi demo"
          size="sm"
        />
      </template>
    </UHeader>

    <UMain>
      <NuxtPage />
    </UMain>

    <UFooter>
      <template #left>
        <p class="text-sm text-muted">
          © {{ new Date().getFullYear() }} Agora Games. Tutti i diritti riservati.
        </p>
      </template>

      <template #center>
        <UNavigationMenu
          :items="[
            { label: 'Docs', to: '/docs' },
            { label: 'Privacy Policy', to: '/privacy' },
            { label: 'Termini di servizio', to: '/termini' },
            { label: 'hello@agoragames.io', to: 'mailto:hello@agoragames.io' }
          ]"
          variant="link"
          class="text-sm"
        />
      </template>
    </UFooter>
  </UApp>
</template>
