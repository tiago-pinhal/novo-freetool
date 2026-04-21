<script setup lang="ts">
const { t } = useI18n({ useScope: 'local' })
const localePath = useLocalePath()

usePageJsonLd({
  type: 'page',
  name: t('title'),
  description: t('meta'),
  breadcrumb: [
    { name: 'Home', url: localePath('/') },
    { name: t('title') },
  ],
})

useHead({
  title: t('title'),
  meta: [{ name: 'description', content: t('meta') }],
})

defineI18nRoute({
  paths: {
    pt: '/ferramentas-para-datas-e-horas',
    es: '/herramientas-para-fechas-y-horas',
    fr: '/outils-pour-dates-et-heures',
    it: '/strumenti-per-date-e-ore',
    id: '/alat-tanggal-dan-waktu',
    de: '/datum-zeit-werkzeuge',
  },
})
</script>

<template>
  <div class="max-w-6xl mx-auto px-4">
    <div class="mb-12 text-center">
      <h1 class="tools-title">
        {{ t('title') }}
      </h1>
      <p class="text-lg text-base-content/70 mx-auto">
        {{ t('desc') }}
      </p>
    </div>

    <div class="flex flex-wrap justify-center gap-6">
      <ToolCard
        class="w-full sm:w-[calc(50%-12px)] lg:w-[calc(25%-18px)]"
        :title="t('dt')"
        to="date-difference"
        icon="mdi:calendar-blank"
        :index="0"
      />
      <ToolCard
        class="w-full sm:w-[calc(50%-12px)] lg:w-[calc(25%-18px)]"
        :title="t('times')"
        to="time-difference"
        icon="mdi:clock-time-three-outline"
        :index="1"
      />
      <ToolCard
        class="w-full sm:w-[calc(50%-12px)] lg:w-[calc(25%-18px)]"
        :title="t('diff')"
        to="date-time-difference"
        icon="mdi:calendar-clock"
        :index="2"
      />
      <ToolCard
        class="w-full sm:w-[calc(50%-12px)] lg:w-[calc(25%-18px)]"
        :title="t('birth')"
        to="birthday-generator"
        icon="mdi:calendar-account-outline"
        :index="3"
      />
      <ToolCard
        class="w-full sm:w-[calc(50%-12px)] lg:w-[calc(25%-18px)]"
        :title="t('age')"
        to="age-calculator"
        icon="mdi:calculator"
        :index="4"
      />
      <ToolCard
        class="w-full sm:w-[calc(50%-12px)] lg:w-[calc(25%-18px)]"
        :title="t('hour')"
        to="hours-and-minutes-calculator"
        icon="mdi:alarm-plus"
        :index="5"
      />
      <ToolCard
        class="w-full sm:w-[calc(50%-12px)] lg:w-[calc(25%-18px)]"
        :title="t('future')"
        to="future-date-calculator"
        icon="mdi:calendar-plus"
        :index="6"
      />
      <ToolCard
        class="w-full sm:w-[calc(50%-12px)] lg:w-[calc(25%-18px)]"
        :title="t('past')"
        to="past-date-calculator"
        icon="mdi:calendar-minus"
        :index="7"
      />
      <ToolCard
        class="w-full sm:w-[calc(50%-12px)] lg:w-[calc(25%-18px)]"
        :title="t('time')"
        to="time-converter"
        icon="mdi:arrow-left-right"
        :index="8"
      />
    </div>

    <div class="flex items-center gap-4 my-12">
      <span class="text-base font-semibold text-base-content/50 whitespace-nowrap">{{ t('devs') }}</span>
      <div class="h-px flex-1 bg-base-content/10" />
    </div>

    <div class="flex flex-wrap justify-center gap-6">
      <ToolCard
        class="w-full sm:w-[calc(50%-12px)] lg:w-[calc(25%-18px)]"
        :title="t('cron')"
        to="crontab-generator"
        icon="mdi:asterisk"
        :index="9"
      />
      <ToolCard
        class="w-full sm:w-[calc(50%-12px)] lg:w-[calc(25%-18px)]"
        :title="t('unix')"
        to="unix-timestamp-converter"
        icon="mdi:clock-outline"
        :index="10"
      />
    </div>

    <div class="mt-16 bg-base-200/50 rounded-3xl p-8 border border-base-content/5">
      <Adsense />
    </div>
  </div>
</template>

<i18n lang="yaml">
{
    en: {
        title: "Dates and Times Tools",
        meta: "Free online date and time tools: calculate differences, generate dates, age calculator, crontab generator and more. Simplify time calculations!",
        desc: "Calculate date and time differences, generate random dates, find out someone's age, convert time units or create cron expressions and Unix timestamps. All processing happens in your browser, your data never leaves your device and no installation is required.",
        devs: "For Developers",
        dt: "Date Difference",
        times: "Time Difference",
        diff: "Date Time Difference",
        birth: "Birthday Generator",
        age: "Age Calculator",
        hour: "Hours and Minutes Calculator",
        future: "Future Date Calculator",
        past: "Past Date Calculator",
        time: "Time Converter",
        cron: "Crontab Generator",
        unix: "Unix Timestamp Converter",
    },
    pt: {
        title: "Ferramentas para Datas e Horas",
        meta: "Ferramentas de data e hora online grátis: calcular diferenças, gerar datas, calculadora de idade, gerador de crontab e mais. Simplifique cálculos de tempo!",
        desc: "Calcule diferenças entre datas e horas, gere datas aleatórias, descubra a idade de alguém, converta unidades de tempo ou crie expressões cron e converta timestamps Unix. Todo o processamento acontece no seu navegador, seus dados nunca saem do seu dispositivo e nenhuma instalação é necessária.",
        devs: "Para Desenvolvedores",
        dt: "Diferença entre Datas",
        times: "Diferença entre Horas",
        diff: "Diferença entre Datas e Horas",
        birth: "Gerador de Data de Nascimento",
        age: "Calculadora de Idade",
        hour: "Calculadora de Horas e Minutos",
        future: "Calculadora de Data Futura",
        past: "Calculadora de Data Passada",
        time: "Conversor de Tempo",
        cron: "Gerador de Crontab",
        unix: "Conversor de Timestamp Unix",
    },
    es: {
        title: "Herramientas para Fechas y Horas",
        meta: "Herramientas de fecha y hora online gratis: calcular diferencias, generar fechas, calculadora de edad, generador de crontab y más. ¡Simplifica cálculos de tiempo!",
        desc: "Calcula diferencias entre fechas y horas, genera fechas aleatorias, averigua la edad de alguien, convierte unidades de tiempo o crea expresiones cron y convierte timestamps Unix. Todo el procesamiento ocurre en tu navegador, tus datos nunca salen de tu dispositivo y no necesitas instalar nada.",
        devs: "Para Desarrolladores",
        dt: "Diferencia entre Fechas",
        times: "Diferencia entre Horas",
        diff: "Diferencia entre Fechas y Horas",
        birth: "Generador de Fecha de Nacimiento",
        age: "Calculadora de Edad",
        hour: "Calculadora de Horas y Minutos",
        future: "Calculadora de Fecha Futura",
        past: "Calculadora de Fecha Pasada",
        time: "Convertidor de Tiempo",
        cron: "Generador de Crontab",
        unix: "Convertidor de Timestamp Unix",
    },
    fr: {
        title: "Outils pour Dates et Heures",
        meta: "Outils de date et heure en ligne gratuits : calculer les différences, générer des dates, calculateur d'âge, générateur de crontab et plus. Simplifiez les calculs de temps !",
        desc: "Calculez les différences entre dates et heures, générez des dates aléatoires, découvrez l'âge de quelqu'un, convertissez des unités de temps ou créez des expressions cron et convertissez des timestamps Unix. Tout le traitement se fait dans votre navigateur, vos données ne quittent jamais votre appareil et aucune installation n'est requise.",
        devs: "Pour les Développeurs",
        dt: "Différence entre Dates",
        times: "Différence entre Heures",
        diff: "Différence entre Dates et Heures",
        birth: "Générateur de Date de Naissance",
        age: "Calculateur d'Âge",
        hour: "Calculatrice d'Heures et de Minutes",
        future: "Calculatrice de Date Future",
        past: "Calculatrice de Date Passée",
        time: "Convertisseur de Temps",
        cron: "Générateur de Crontab",
        unix: "Convertisseur de Timestamp Unix",
    },
    it: {
        title: "Strumenti per Date e Ore",
        meta: "Strumenti di data e ora online gratuiti: calcolare differenze, generare date, calcolatrice dell'età, generatore di crontab e altro. Semplifica i calcoli temporali!",
        desc: "Calcola le differenze tra date e ore, genera date casuali, scopri l'età di qualcuno, converti unità di tempo o crea espressioni cron e converti timestamp Unix. Tutta l'elaborazione avviene nel tuo browser, i tuoi dati non lasciano mai il dispositivo e non è necessaria alcuna installazione.",
        devs: "Per Sviluppatori",
        dt: "Differenza tra Date",
        times: "Differenza tra Ore",
        diff: "Differenza tra Date e Ore",
        birth: "Generatore di Data di Nascita",
        age: "Calcolatrice dell'Età",
        hour: "Calcolatrice di Ore e Minuti",
        future: "Calcolatrice di Data Futura",
        past: "Calcolatrice di Data Passata",
        time: "Convertitore di Tempo",
        cron: "Generatore di Crontab",
        unix: "Convertitore di Timestamp Unix",
    },
    id: {
        title: "Alat Tanggal dan Waktu",
        meta: "Alat tanggal dan waktu online gratis: hitung perbedaan, buat tanggal, kalkulator usia, generator crontab, dan lainnya. Sederhanakan perhitungan waktu!",
        desc: "Hitung perbedaan tanggal dan waktu, buat tanggal acak, cari tahu usia seseorang, konversi satuan waktu atau buat ekspresi cron dan konversi timestamp Unix. Semua pemrosesan dilakukan di browser Anda, data Anda tidak pernah meninggalkan perangkat dan tidak diperlukan instalasi apa pun.",
        devs: "Untuk Developer",
        dt: "Selisih Tanggal",
        times: "Selisih Waktu",
        diff: "Selisih Tanggal dan Waktu",
        birth: "Generator Tanggal Lahir",
        age: "Kalkulator Usia",
        hour: "Kalkulator Jam dan Menit",
        future: "Kalkulator Tanggal Masa Depan",
        past: "Kalkulator Tanggal Masa Lalu",
        time: "Konverter Waktu",
        cron: "Generator Crontab",
        unix: "Konverter Timestamp Unix",
    },
    de: {
        title: "Datum- und Zeitwerkzeuge",
        meta: "Kostenlose Online-Datum- und Zeittools: Unterschiede berechnen, Daten generieren, Altersrechner, Crontab-Generator und mehr. Vereinfachen Sie Zeitberechnungen!",
        desc: "Berechnen Sie Datums- und Zeitunterschiede, generieren Sie zufällige Daten, ermitteln Sie das Alter einer Person, konvertieren Sie Zeiteinheiten oder erstellen Sie Cron-Ausdrücke und konvertieren Sie Unix-Zeitstempel. Die Verarbeitung erfolgt in Ihrem Browser, Ihre Daten verlassen niemals Ihr Gerät und eine Installation ist nicht erforderlich.",
        devs: "Für Entwickler",
        dt: "Datumsunterschied",
        times: "Zeitunterschied",
        diff: "Datums- und Zeitunterschied",
        birth: "Geburtsdatum-Generator",
        age: "Altersrechner",
        hour: "Stunden- und Minutenrechner",
        future: "Zukünftiges-Datum-Rechner",
        past: "Vergangenes-Datum-Rechner",
        time: "Zeitkonverter",
        cron: "Crontab-Generator",
        unix: "Unix-Zeitstempel-Konverter",
    }
}
</i18n>
