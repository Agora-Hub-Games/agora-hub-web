<template>
  <UPage>
    <UPageHeader
      title="Documentazione"
      description="Guide tecniche per integrare e gestire Agora Games."
    />
    <UPageBody>
      <div class="grid gap-4 sm:grid-cols-2">
        <UCard
          v-for="doc in docs"
          :key="doc.path"
          :to="doc.path"
          as="NuxtLink"
          class="hover:ring-primary-500 cursor-pointer transition-shadow hover:ring-2"
        >
          <template #header>
            <div class="flex items-center gap-3">
              <UIcon
                v-if="doc.navigation?.icon"
                :name="doc.navigation.icon"
                class="text-primary size-5"
              />
              <h3 class="font-semibold">{{ doc.title }}</h3>
            </div>
          </template>
          <p class="text-muted text-sm">{{ doc.description }}</p>
        </UCard>
      </div>
    </UPageBody>
  </UPage>
</template>

<script setup lang="ts">
const { data: docs } = await useAsyncData('docs-index', () =>
  queryCollection('docs').all()
)
</script>
