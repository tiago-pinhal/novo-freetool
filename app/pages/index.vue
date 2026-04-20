<script setup lang="ts">
const { t } = useI18n({ useScope: 'local' })

useHead({
  title: 'Home | Freetool.Dev',
  meta: [{ name: "description", content: t('d2') }],
})

const toolCategories = [
  { id: 'pdf', to: '/pdf-tools', icon: 'document', title: t('pdf') || 'PDF Tools' },
  { id: 'color', to: '/color-tools', icon: 'swatch', title: t('clr') },
  { id: 'image', to: '/image-tools', icon: 'photo', title: t('img') },
  { id: 'text', to: '/text-tools', icon: 'document-text', title: t('txt') },
  { id: 'datetime', to: '/date-time-tools', icon: 'calendar', title: t('dts') },
  { id: 'random', to: '/random-data', icon: 'circle-stack', title: t('rand') },
  { id: 'minify', to: '/code-minifiers', icon: 'arrows-pointing-in', title: t('min') },
  { id: 'format', to: '/code-formatters', icon: 'bars-4', title: t('fmt') },
  { id: 'convert', to: '/converters', icon: 'arrows-right-left', title: t('conv') },
  { id: 'misc', to: '/various-tools', icon: 'squares-2x2', title: t('misc') },
]
</script>

<template>
  <div class="min-h-screen bg-base-100">
    <!-- Hero Section -->
    <section class="relative overflow-hidden pt-20 pb-16 sm:pt-32 sm:pb-24">
      <!-- Background Glow -->
      <div class="absolute top-0 left-1/2 -translate-x-1/2 w-full max-w-6xl h-full pointer-events-none opacity-20">
        <div class="absolute top-[-10%] left-[-10%] w-[40%] h-[40%] bg-primary rounded-full blur-[120px]"></div>
        <div class="absolute bottom-[10%] right-[-5%] w-[30%] h-[30%] bg-secondary rounded-full blur-[100px]"></div>
      </div>

      <div class="container mx-auto px-4 relative z-10">
        <div class="text-center max-w-4xl mx-auto">
          <h1 class="text-5xl sm:text-8xl font-extrabold tracking-tighter mb-8 leading-tight">
            <span class="bg-gradient-to-r from-emerald-400 via-teal-400 to-cyan-500 bg-clip-text text-transparent">
              {{ t('d1') }} Freetool
            </span>
          </h1>
          <p class="text-lg sm:text-xl text-base-content/70 leading-relaxed mb-10 max-w-2xl mx-auto">
            {{ t('d2') }}
          </p>
          
        </div>
      </div>
    </section>

    <!-- Tools Grid Section -->
    <section class="pb-24 px-4">
      <div class="container mx-auto">
        <div class="flex items-center gap-4 mb-12">
          <h2 class="text-2xl font-bold text-base-content">{{ t('d3') }}</h2>
          <div class="h-px flex-1 bg-base-content/10"></div>
        </div>

        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-5 gap-6">
          <NuxtLinkLocale
            v-for="tool in toolCategories"
            :key="tool.id"
            :to="tool.to"
            class="group relative p-8 rounded-3xl bg-base-200/50 backdrop-blur-sm border border-base-content/5 hover:border-primary/50 transition-all duration-300 hover:-translate-y-2 hover:shadow-2xl hover:shadow-primary/10"
          >
            <div class="flex flex-col items-center text-center">
              <div class="w-16 h-16 rounded-2xl bg-gradient-to-br from-primary/10 to-secondary/10 flex items-center justify-center mb-6 group-hover:from-primary/20 group-hover:to-secondary/20 transition-colors">
                <Icon :name="tool.icon" class="w-8 h-8 text-primary group-hover:scale-110 transition-transform" />
              </div>
              <h3 class="text-xl font-bold text-base-content mb-2 group-hover:text-primary transition-colors">
                {{ tool.title }}
              </h3>
              <div class="absolute top-4 right-4 opacity-0 group-hover:opacity-100 transition-opacity">
                <Icon name="arrow-up-right" class="w-5 h-5 text-base-content/30" />
              </div>
            </div>
          </NuxtLinkLocale>
        </div>
      </div>
    </section>
  </div>
</template>

<i18n lang="yaml">
{
    en: {
        d1: "Welcome to",
        d2: "Explore our platform dedicated to offering free online tools to simplify your daily tasks. We also feature exclusive tools for software developers (DEV), aimed at streamlining your professional activities",
        d3: "Tools available for",
        pdf: "PDF",
        clr: "Colors",
        img: "Images",
        txt: "Texts",
        dts: "Dates and Times",
        rand: "Random Data",
        min: "Minify Code",
        fmt: "Format Code",
        conv: "Converters",
        misc: "Various Tools"
    },
    pt: {
        d1: "Bem-vindo ao",
        d2: "Explore nossa plataforma dedicada a oferecer ferramentas online gratuitas para facilitar suas tarefas cotidianas. Contamos também com ferramentas exclusivas para desenvolvedores de software (DEV), visando agilizar suas atividades profissionais",
        d3: "Ferramentas disponíveis para",
        pdf: "PDF",
        clr: "Cores",
        img: "Imagens",
        txt: "Textos",
        dts: "Datas e Horas",
        rand: "Dados Aleatórios",
        min: "Minificar Código",
        fmt: "Formatar Código",
        conv: "Conversores",
        misc: "Ferramentas Diversas"
    },
    es: {
        d1: "Bienvenido a",
        d2: "Explore nuestra plataforma dedicada a ofrecer herramientas en línea gratuitas para facilitar sus tareas diarias. También contamos con herramientas exclusivas para desarrolladores de software (DEV), destinadas a agilizar sus actividades profesionales",
        d3: "Herramientas disponibles para",
        pdf: "PDF",
        clr: "Colores",
        img: "Imágenes",
        txt: "Textos",
        dts: "Fechas y Horas",
        rand: "Datos Aleatorios",
        min: "Minificar Código",
        fmt: "Formatear Código",
        conv: "Convertidores",
        misc: "Herramientas Diversas"
    },
    fr: {
        d1: "Bienvenue chez",
        d2: "Explorez notre plateforme dédiée à offrir des outils en ligne gratuits pour simplifier vos tâches quotidiennes. Nous proposons également des outils exclusifs pour les développeurs de logiciels (DEV), visant à optimiser leurs activités professionnelles",
        d3: "Outils disponibles pour",
        pdf: "PDF",
        clr: "Couleurs",
        img: "Images",
        txt: "Textes",
        dts: "Dates et Heures",
        rand: "Données Aléatoires",
        min: "Minifier le Code",
        fmt: "Formater le Code",
        conv: "Convertisseurs",
        misc: "Outils Divers"
    },
    it: {
        d1: "Benvenuto su",
        d2: "Esplora la nostra piattaforma dedicata a offrire strumenti online gratuiti per facilitare i tuoi compiti quotidiani. Offriamo inoltre strumenti esclusivi per sviluppatori di software (DEV), mirati a velocizzare le tue attività professionali",
        d3: "Strumenti disponibili per",
        pdf: "PDF",
        clr: "Colori",
        img: "Immagini",
        txt: "Testi",
        dts: "Date e Ore",
        rand: "Dati Casuali",
        min: "Minimizza Codice",
        fmt: "Formatta Codice",
        conv: "Convertitori",
        misc: "Strumenti Diversi"
    },
    id: {
        d1: "Selamat datang di",
        d2: "Jelajahi platform kami yang menyediakan alat online gratis untuk mempermudah tugas sehari-hari Anda. Kami juga menghadirkan alat ekklusif untuk pengembang perangkat lunak (DEV), dirancang untuk meningkatkan efisiensi aktivitas profesional Anda",
        d3: "Alat tersedia untuk",
        pdf: "PDF",
        clr: "Warna",
        img: "Gambar",
        txt: "Teks",
        dts: "Tanggal dan Waktu",
        rand: "Data Acak",
        min: "Minifikasi Kode",
        fmt: "Format Kode",
        conv: "Konverter",
        misc: "Alat Lainnya"
    }
}
</i18n>
