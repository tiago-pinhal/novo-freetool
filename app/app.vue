<script setup lang="ts">
const head = useLocaleHead({ dir: true, lang: true, seo: true })

useHead({
  htmlAttrs: {
    lang: () => head.value.htmlAttrs?.lang,
    dir: () => head.value.htmlAttrs?.dir as 'ltr' | 'rtl' | 'auto' | undefined
  },
  link: () => head.value.link || [],
  meta: () => head.value.meta || []
})
</script>

<template>
  <NuxtLayout>
    <NuxtPage />
  </NuxtLayout>
</template>