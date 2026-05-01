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
  en: { title: "Privacy Policy", desc: "This page is under construction. Soon you will find our Privacy Policy here." },
  pt: { title: "Política de Privacidade", desc: "Esta página está em construção. Em breve aqui você encontrará nossa Política de Privacidade." },
  es: { title: "Política de Privacidad", desc: "Esta página está en construcción. Pronto encontrarás aqui nuestra Política de Privacidad." },
  fr: { title: "Politique de Confidentialité", desc: "Cette page est en construction. Bientôt, vous trouverez ici notre Politique de Confidentialité." },
  it: { title: "Informativa sulla Privacy", desc: "Questa pagina è in costruzione. Presto troverai qui la nostra Informativa sulla Privacy." },
  id: { title: "Kebijakan Privasi", desc: "Halaman ini sedang dalam pembangunan. Segera Anda akan menemukan Kebijakan Privasi kami di sini." },
  de: { title: "Datenschutzbestimmungen", desc: "Diese Seite befindet sich im Aufbau. In Kürze findest du hier unsere Datenschutzbestimmungen." },
  nl: { title: "Privacybeleid", desc: "Deze pagina is in aanbouw. Binnenkort vind je hier ons privacybeleid." }
}
</i18n>
