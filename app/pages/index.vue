<template>
  <div>
    <!-- Hero -->
    <UPageHero
      title="Trasforma i visitatori in clienti fedeli. Con il gioco."
      description="Agora Games è il widget di gamification che aggiungi al tuo sito in 5 minuti: i tuoi clienti giocano, vincono coupon e tornano ad acquistare. Zero sviluppo richiesto."
      :links="[
        {
          label: 'Richiedi demo',
          to: 'https://admin.agoragames.io/signup',
          target: '_blank',
          trailingIcon: 'i-lucide-arrow-right',
          size: 'xl'
        },
        {
          label: 'Leggi la documentazione',
          to: '/docs',
          size: 'xl',
          color: 'neutral',
          variant: 'subtle'
        }
      ]"
    />

    <!-- Features -->
    <UPageSection
      id="features"
      title="Tutto ciò che serve. Niente di superfluo."
      description="Sei funzionalità pensate per chi vuole più conversioni senza complicarsi la vita."
      :features="[
        {
          icon: 'i-lucide-code-2',
          title: 'Zero sviluppo richiesto',
          description: 'Un solo tag &lt;script&gt; sul tuo sito e il widget è live. Nessuna integrazione backend, nessun deploy.'
        },
        {
          icon: 'i-lucide-gamepad-2',
          title: 'Gamification configurabile',
          description: 'Slot machine, ruota della fortuna, gratta e vinci: scegli il gioco e brandizzalo con i tuoi colori e loghi.'
        },
        {
          icon: 'i-lucide-bar-chart-2',
          title: 'Analytics & tracking eventi',
          description: 'Monitora sessioni, vincite, conversioni coupon e comportamento utente in tempo reale dal pannello admin.'
        },
        {
          icon: 'i-lucide-layers',
          title: 'Multi-brand e multi-gioco',
          description: 'Gestisci più brand e più siti da un unico pannello. Ogni brand ha configurazione, giochi e analytics indipendenti.'
        },
        {
          icon: 'i-lucide-smartphone',
          title: 'Mobile-first',
          description: 'Esperienza fluida su qualsiasi dispositivo. Il widget si adatta perfettamente agli schermi mobile e tablet.'
        },
        {
          icon: 'i-lucide-ticket',
          title: 'Coupon e lead generation',
          description: 'Genera coupon automatici personalizzati, raccogli email e integra il tutto con il tuo e-commerce o CRM.'
        }
      ]"
    />

    <!-- How it works -->
    <UPageSection
      id="come-funziona"
      title="Online in tre passi"
      description="Dalla registrazione al widget live: meno di un'ora."
    >
      <div class="grid grid-cols-1 gap-8 md:grid-cols-3 mt-8">
        <div
          v-for="(step, index) in steps"
          :key="index"
          class="flex flex-col items-start gap-4"
        >
          <div class="flex items-center justify-center w-12 h-12 rounded-full bg-primary text-white font-bold text-lg shrink-0">
            {{ index + 1 }}
          </div>
          <div>
            <h3 class="text-base font-semibold text-highlighted mb-1">
              {{ step.title }}
            </h3>
            <p class="text-sm text-muted">
              {{ step.description }}
            </p>
          </div>
        </div>
      </div>
    </UPageSection>

    <!-- Pricing -->
    <UPageSection
      id="pricing"
      title="Piani semplici e trasparenti"
      description="Inizia gratis con il piano Starter. Scala quando sei pronto."
    >
      <div class="grid grid-cols-1 gap-6 md:grid-cols-3 mt-8">
        <UCard
          v-for="plan in plans"
          :key="plan.name"
          :class="['flex flex-col', plan.highlighted ? 'ring-2 ring-primary' : '']"
        >
          <template #header>
            <div class="flex items-start justify-between gap-2">
              <div>
                <p class="text-sm font-semibold text-muted uppercase tracking-wide">
                  {{ plan.name }}
                </p>
                <div class="flex items-end gap-1 mt-1">
                  <span class="text-3xl font-bold text-highlighted">{{ plan.price }}</span>
                  <span v-if="plan.period" class="text-sm text-muted mb-1">{{ plan.period }}</span>
                </div>
                <p class="text-sm text-muted mt-1">
                  {{ plan.tagline }}
                </p>
              </div>
              <UBadge v-if="plan.highlighted" label="Consigliato" color="primary" variant="subtle" />
            </div>
          </template>

          <ul class="flex flex-col gap-2 grow">
            <li
              v-for="feature in plan.features"
              :key="feature"
              class="flex items-start gap-2 text-sm text-default"
            >
              <UIcon name="i-lucide-check" class="text-primary mt-0.5 shrink-0" />
              {{ feature }}
            </li>
          </ul>

          <template #footer>
            <UButton
              :to="plan.cta.to"
              :target="plan.cta.external ? '_blank' : undefined"
              :label="plan.cta.label"
              :color="plan.highlighted ? 'primary' : 'neutral'"
              :variant="plan.highlighted ? 'solid' : 'outline'"
              block
            />
          </template>
        </UCard>
      </div>

      <p class="text-center text-sm text-muted mt-6">
        Hai bisogno di più sessioni? Sono disponibili pacchetti di visite aggiuntivi per ogni piano.
        <NuxtLink to="mailto:hello@agoragames.io" class="text-primary underline">Contattaci</NuxtLink>.
      </p>
    </UPageSection>

    <!-- Final CTA -->
    <UPageSection>
      <UPageCTA
        title="Pronto a far giocare i tuoi clienti?"
        description="Inizia oggi. Nessuna carta di credito richiesta per la demo."
        variant="subtle"
        :links="[
          {
            label: 'Richiedi demo',
            to: 'https://admin.agoragames.io/signup',
            target: '_blank',
            trailingIcon: 'i-lucide-arrow-right'
          },
          {
            label: 'Hai domande? Scrivici',
            to: 'mailto:hello@agoragames.io',
            color: 'neutral',
            variant: 'outline'
          }
        ]"
      />
    </UPageSection>
  </div>
</template>

<script setup lang="ts">
const steps = [
  {
    title: 'Configura il tuo brand in dashboard',
    description: 'Accedi al pannello, scegli il tipo di gioco, carica i tuoi colori e loghi, configura i premi e le probabilità di vincita.'
  },
  {
    title: 'Copia lo snippet e incollalo sul tuo sito',
    description: 'Un solo tag <script> da incollare prima del </body>. Funziona su qualsiasi CMS o piattaforma e-commerce.'
  },
  {
    title: 'I tuoi clienti giocano e vincono coupon',
    description: 'I visitatori giocano, vincono coupon personalizzati e tornano più motivati ad acquistare. Tu monitora tutto in real-time.'
  }
]

const plans = [
  {
    name: 'Starter',
    price: '€49',
    period: '/ mese',
    tagline: 'Perfetto per iniziare',
    highlighted: false,
    features: [
      'Fino a 5.000 sessioni / mese',
      '1 brand',
      'Giochi base (slot, ruota)',
      'Analytics standard',
      'Coupon automatici',
      'Supporto via email'
    ],
    cta: { label: 'Inizia con Starter', to: 'https://admin.agoragames.io/signup', external: true }
  },
  {
    name: 'Pro',
    price: '€149',
    period: '/ mese',
    tagline: 'Per chi vuole crescere',
    highlighted: true,
    features: [
      'Fino a 25.000 sessioni / mese',
      'Fino a 3 brand',
      'Tutti i tipi di gioco',
      'Analytics avanzati e export',
      'Lead generation & raccolta email',
      'Supporto prioritario'
    ],
    cta: { label: 'Inizia con Pro', to: 'https://admin.agoragames.io/signup', external: true }
  },
  {
    name: 'Enterprise',
    price: 'Su misura',
    period: '',
    tagline: 'Per grandi volumi e team',
    highlighted: false,
    features: [
      'Sessioni illimitate',
      'Brand illimitati',
      'SLA dedicato',
      'Integrazioni custom',
      'Onboarding assistito',
      'Account manager dedicato'
    ],
    cta: { label: 'Contattaci', to: 'mailto:hello@agoragames.io', external: false }
  }
]
</script>
