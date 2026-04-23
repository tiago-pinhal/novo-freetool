<script setup lang="ts">
const { t } = useI18n()

useHead({
  title: `${t('title')} | Freetool.Dev`,
})

usePageJsonLd({
  type: 'page',
  name: `${t('title')} | Freetool.Dev`,
})
</script>

<template>
  <div class="container mx-auto px-4 py-24 min-h-[60vh]">
    <div class="max-w-3xl mx-auto">
      <h1 class="text-4xl font-extrabold mb-8 bg-gradient-to-r from-primary to-secondary bg-clip-text text-transparent">
        {{ t('title') }} Freetool
      </h1>
      <div class="prose lg:prose-xl opacity-80">
        <p>{{ t('desc') }}</p>
      </div>
    </div>
  </div>
</template>

<i18n lang="yaml">
{
  en: { title: "About", desc: "This page is under construction. Soon you will find more information about the Freetool project here." },
  pt: { title: "Sobre", desc: "Esta página está em construção. Em breve aqui você encontrará mais informações sobre o projeto Freetool." },
  es: { title: "Sobre", desc: "Esta página está en construcción. Pronto encontrarás más información sobre o proyecto Freetool aqui." },
  fr: { title: "Propos", desc: "Cette page est en construction. Bientôt, vous trouverez ici plus d'informations sur le projet Freetool." },
  it: { title: "Su di noi", desc: "Questa pagina è in costruzione. Presto troverai qui maggiori informazioni sul progetto Freetool." },
  id: { title: "Tentang Kami", desc: "Halaman ini sedang dalam pembangunan. Segera Anda akan menemukan informasi lebih lanjut tentang proyek Freetool di sini." }
}
</i18n>
