<template>
  <UPage v-if="doc">
    <UPageHeader :title="doc.title" :description="doc.description" />
    <UPageBody prose>
      <ContentRenderer :value="doc" />
    </UPageBody>
  </UPage>
  <UPage v-else>
    <UPageError :error="{ statusCode: 404, message: 'Pagina non trovata' }" />
  </UPage>
</template>

<script setup lang="ts">
const route = useRoute()
const slug = Array.isArray(route.params.slug) ? route.params.slug.join('/') : route.params.slug

const { data: doc } = await useAsyncData(`doc-${slug}`, () =>
  queryCollection('docs').path(`/docs/${slug}`).first()
)

if (!doc.value) {
  throw createError({ statusCode: 404, statusMessage: 'Pagina non trovata' })
}

useSeoMeta({
  title: doc.value?.title,
  description: doc.value?.description
})
</script>
