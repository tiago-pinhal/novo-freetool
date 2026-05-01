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
        {{ t('title') }}
      </h1>
      <div class="prose lg:prose-xl opacity-80">
        <p>{{ t('desc') }}</p>
      </div>
    </div>
  </div>
</template>

<i18n lang="yaml">
{
  en: { title: "Terms of Use", desc: "This page is under construction. Soon you will find our Terms of Use here." },
  pt: { title: "Termos de Uso", desc: "Esta página está em construção. Em breve aqui você encontrará nossos Termos de Uso." },
  es: { title: "Términos de Uso", desc: "Esta página está en construcción. Pronto encontrarás aqui nossos Términos de Uso." },
  fr: { title: "Conditions d'Utilisation", desc: "Cette page est en construction. Bientôt, vous trouverez ici nos Conditions d'Utilisation." },
  it: { title: "Termini di Utilizzo", desc: "Questa pagina è in costruzione. Presto troverai qui i nostri Termini di Utilizzo." },
  id: { title: "Ketentuan Penggunaan", desc: "Halaman ini sedang dalam pembangunan. Segera Anda akan menemukan Ketentuan Penggunaan kami di sini." },
  de: { title: "Nutzungsbedingungen", desc: "Diese Seite befindet sich im Aufbau. In Kürze findest du hier unsere Nutzungsbedingungen." },
  nl: { title: "Gebruiksvoorwaarden", desc: "Deze pagina is in aanbouw. Binnenkort vind je hier onze gebruiksvoorwaarden." }
}
</i18n>
