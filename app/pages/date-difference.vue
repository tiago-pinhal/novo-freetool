<script setup lang="ts">
const { t } = useI18n({ useScope: 'local' })

interface Result {
  years: number
  months: number
  days: number
  totalDays: number
  totalWeeks: number
  totalMonths: number
  totalHours: number
  totalMinutes: number
}

const state = reactive({
  p1: '',
  p2: '',
  result: null as Result | null,
  equal: false,
})

function parseDate(s: string): Date {
  const [y, m, d] = s.split('-').map(Number)
  return new Date(y, m - 1, d)
}

function calc() {
  state.result = null
  state.equal = false

  if (!state.p1 || !state.p2) return

  let d1 = parseDate(state.p1)
  let d2 = parseDate(state.p2)

  if (d1.getTime() === d2.getTime()) {
    state.equal = true
    return
  }

  if (d1 > d2) [d1, d2] = [d2, d1]

  let years = d2.getFullYear() - d1.getFullYear()
  let months = d2.getMonth() - d1.getMonth()
  let days = d2.getDate() - d1.getDate()

  if (days < 0) {
    months--
    days += new Date(d2.getFullYear(), d2.getMonth(), 0).getDate()
  }

  if (months < 0) {
    years--
    months += 12
  }

  const diffMs = d2.getTime() - d1.getTime()
  const totalDays = Math.round(diffMs / 86400000)

  state.result = {
    years,
    months,
    days,
    totalDays,
    totalWeeks: Math.floor(totalDays / 7),
    totalMonths: years * 12 + months,
    totalHours: Math.round(diffMs / 3600000),
    totalMinutes: Math.round(diffMs / 60000),
  }
}

watch(() => state.p1, calc)
watch(() => state.p2, calc)

usePageJsonLd({
  name: t('title'),
  description: t('meta'),
  features: [t('f_1'), t('f_2'), t('f_3'), t('f_4'), t('f_5')],
  faq: [
    { question: t('faq1q'), answer: t('faq1a') },
    { question: t('faq2q'), answer: t('faq2a') },
    { question: t('faq3q'), answer: t('faq3a') },
    { question: t('faq4q'), answer: t('faq4a') },
  ],
  howToName: t('how_to_use_title'),
  howToSteps: [
    { name: t('step_1_title'), text: t('step_1_desc') },
    { name: t('step_2_title'), text: t('step_2_desc') },
    { name: t('step_3_title'), text: t('step_3_desc') },
  ],
})

useHead({
  title: t('pageTitle'),
  meta: [{ name: 'description', content: t('meta') }],
})

defineI18nRoute({
  paths: {
    pt: '/diferenca-entre-datas',
    es: '/diferencia-entre-fechas',
    fr: '/difference-entre-dates',
    it: '/differenza-tra-date',
    id: '/selisih-tanggal',
  },
})
</script>

<template>
  <ToolPage
    :title="t('title')"
    :description="t('meta')"
    :see-also-links="[
      { label: t('see1'), to: 'time-difference' },
      { label: t('see2'), to: 'date-time-difference' },
      { label: t('see3'), to: 'future-date-calculator' },
      { label: t('see4'), to: 'past-date-calculator' },
    ]"
  >
    <!-- Inputs -->
    <div class="flex flex-wrap gap-4 mb-6">
      <div class="form-control flex-1 min-w-44">
        <label class="label pb-1" for="p1">
          <span class="label-text font-bold text-base-content/80">{{ t('param1') }}</span>
        </label>
        <input
          id="p1"
          type="date"
          v-model="state.p1"
          :max="state.p2 || undefined"
          class="input input-bordered w-full"
        />
      </div>
      <div class="form-control flex-1 min-w-44">
        <label class="label pb-1" for="p2">
          <span class="label-text font-bold text-base-content/80">{{ t('param2') }}</span>
        </label>
        <input
          id="p2"
          type="date"
          v-model="state.p2"
          :min="state.p1 || undefined"
          class="input input-bordered w-full"
        />
      </div>
    </div>

    <!-- Equal dates warning -->
    <div v-if="state.equal" class="alert alert-warning">
      <Icon name="heroicons:exclamation-triangle" class="w-5 h-5 shrink-0" />
      <span>{{ t('eq') }}</span>
    </div>

    <!-- Results -->
    <div v-if="state.result" class="space-y-4">
      <!-- Exact breakdown -->
      <div>
        <p class="text-xs uppercase tracking-widest text-base-content/50 mb-2">{{ t('diff') }}</p>
        <div class="stats shadow w-fit border border-base-content/10">
          <div v-if="state.result.years > 0" class="stat text-center">
            <div class="stat-title">{{ t('years_label') }}</div>
            <div class="stat-value text-primary">{{ state.result.years }}</div>
          </div>
          <div v-if="state.result.months > 0" class="stat text-center">
            <div class="stat-title">{{ t('months_label') }}</div>
            <div class="stat-value text-primary">{{ state.result.months }}</div>
          </div>
          <div v-if="state.result.days > 0" class="stat text-center">
            <div class="stat-title">{{ t('days_label') }}</div>
            <div class="stat-value text-primary">{{ state.result.days }}</div>
          </div>
        </div>
      </div>

      <!-- Equivalents -->
      <div>
        <p class="text-xs uppercase tracking-widest text-base-content/50 mb-2">{{ t('or') }}</p>
        <div class="stats stats-vertical lg:stats-horizontal shadow w-fit border border-base-content/10">
          <div v-if="state.result.totalMonths > 0" class="stat text-center">
            <div class="stat-title">{{ t('total_months_label') }}</div>
            <div class="stat-value text-3xl">{{ state.result.totalMonths.toLocaleString() }}</div>
          </div>
          <div v-if="state.result.totalWeeks > 0" class="stat text-center">
            <div class="stat-title">{{ t('total_weeks_label') }}</div>
            <div class="stat-value text-3xl">{{ state.result.totalWeeks.toLocaleString() }}</div>
          </div>
          <div class="stat text-center">
            <div class="stat-title">{{ t('total_days_label') }}</div>
            <div class="stat-value text-3xl">{{ state.result.totalDays.toLocaleString() }}</div>
          </div>
          <div v-if="state.result.totalDays <= 730" class="stat text-center">
            <div class="stat-title">{{ t('total_hours_label') }}</div>
            <div class="stat-value text-3xl">{{ state.result.totalHours.toLocaleString() }}</div>
          </div>
          <div v-if="state.result.totalDays <= 60" class="stat text-center">
            <div class="stat-title">{{ t('total_minutes_label') }}</div>
            <div class="stat-value text-3xl">{{ state.result.totalMinutes.toLocaleString() }}</div>
          </div>
        </div>
      </div>
    </div>

    <template #info>
      <div class="space-y-8">
        <section>
          <p class="text-base-content/80 leading-relaxed">{{ t('how_desc') }}</p>
        </section>

        <FeatureSection
          :title="t('features_title')"
          :items="[t('f_1'), t('f_2'), t('f_3'), t('f_4'), t('f_5')]"
        />

        <UseCaseSection
          :title="t('use_cases_title')"
          :items="[t('use_1'), t('use_2'), t('use_3'), t('use_4'), t('use_5'), t('use_6'), t('use_7')]"
        />

        <HowToSection
          :title="t('how_to_use_title')"
          :items="[
            { title: t('step_1_title'), description: t('step_1_desc') },
            { title: t('step_2_title'), description: t('step_2_desc') },
            { title: t('step_3_title'), description: t('step_3_desc') },
          ]"
        />

        <section>
          <h2 class="text-2xl font-bold text-base-content mb-4 flex items-center gap-2">
            <Icon name="heroicons:light-bulb-20-solid" class="text-primary w-6 h-6" aria-hidden="true" />
            {{ t('ex_title') }}
          </h2>
          <ul class="space-y-3 text-base-content/80">
            <li class="flex gap-3">
              <span class="text-primary mt-0.5">•</span>
              <span>{{ t('ex1') }}</span>
            </li>
            <li class="flex gap-3">
              <span class="text-primary mt-0.5">•</span>
              <span>{{ t('ex2') }}</span>
            </li>
            <li class="flex gap-3">
              <span class="text-primary mt-0.5">•</span>
              <span>{{ t('ex3') }}</span>
            </li>
            <li class="flex gap-3">
              <span class="text-primary mt-0.5">•</span>
              <span>{{ t('ex4') }}</span>
            </li>
          </ul>
        </section>

        <FaqSection
          :title="t('faq_title')"
          :items="[
            { question: t('faq1q'), answer: t('faq1a') },
            { question: t('faq2q'), answer: t('faq2a') },
            { question: t('faq3q'), answer: t('faq3a') },
            { question: t('faq4q'), answer: t('faq4a') },
          ]"
        />
      </div>
    </template>
  </ToolPage>
</template>

<i18n lang="yaml">
{
  en: {
    title: "Date Difference Calculator",
    pageTitle: "Date Difference Calculator — Calculate Days Between Two Dates | Free",
    meta: "Calculate the difference between two dates instantly. Get exact results in years, months and days, plus equivalents in weeks, total days, hours and minutes. Free, no registration required.",
    param1: "Start Date",
    param2: "End Date",
    diff: "Exact Difference",
    or: "Or equivalently…",
    years_label: "Years",
    months_label: "Months",
    days_label: "Days",
    total_months_label: "Total Months",
    total_weeks_label: "Total Weeks",
    total_days_label: "Total Days",
    total_hours_label: "Total Hours",
    total_minutes_label: "Total Minutes",
    eq: "Please select two different dates to calculate the difference",
    features_title: "Features",
    f_1: "Exact breakdown in years, months and remaining days",
    f_2: "Equivalent totals: months, weeks, days, hours and minutes",
    f_3: "Automatic leap year handling",
    f_4: "Automatic date order correction",
    f_5: "Free and no registration required",
    how_desc: "Enter a start date and an end date. The calculator instantly shows the elapsed time as an exact breakdown (years, months and days) and as equivalent totals in weeks, total days, hours or minutes depending on the duration. Leap years and months of different lengths are handled automatically.",
    use_cases_title: "Main Applications",
    use_1: "Project Management: track deadlines, milestones and timelines",
    use_2: "Event Planning: schedule events based on specific dates and precise intervals",
    use_3: "Deadline Tracking: stay organised by calculating delivery, meeting or appointment deadlines",
    use_4: "Financial Calculations: determine payment periods, contract expiries and investment durations",
    use_5: "Personal Planning: schedule trips, anniversaries and other commitments",
    use_6: "Education: calculate academic deadlines, school terms and study schedules",
    use_7: "Health & Wellness: track intervals between appointments, treatments or medication cycles",
    how_to_use_title: "How to Use This Tool",
    step_1_title: "Enter a Start Date",
    step_1_desc: "Click the Start Date field and pick the first date from the calendar or type it directly.",
    step_2_title: "Enter an End Date",
    step_2_desc: "Click the End Date field and pick the second date. Results appear instantly.",
    step_3_title: "Read the Results",
    step_3_desc: "The top row shows the exact breakdown in years, months and days. The second row shows convenient equivalent totals.",
    ex_title: "Common Examples",
    ex1: "Holiday planning: from December 20 to January 5 — the calculator shows 16 days, making it easy to request time off or book accommodation.",
    ex2: "Project deadline: from March 1 to June 30 — the result is 3 months and 29 days, useful for sprint and milestone planning.",
    ex3: "Pregnancy tracking: from February 14 to November 14 — exactly 9 months, useful for expectant parents to plan ahead.",
    ex4: "Contract duration: from January 1 to December 31 — 365 days (or 366 in a leap year), common in annual agreements.",
    faq_title: "Questions & Answers",
    faq1q: "Does the calculator count the start date?",
    faq1a: "No. The calculation counts the time elapsed between the two dates. For example, from January 1 to January 3 the result is 2 days — the start date is not included in the count.",
    faq2q: "Does this tool handle leap years correctly?",
    faq2a: "Yes. The calculation correctly recognises leap years. February 2024 has 29 days, and this is automatically reflected in the results.",
    faq3q: "What happens if I enter the end date before the start date?",
    faq3a: "The tool automatically reverses the dates and calculates the difference normally. You will always receive a positive result regardless of the order.",
    faq4q: "Is this date difference calculator free?",
    faq4a: "Yes, 100% free. No registration, no limits and no hidden fees. Use it as many times as you need directly in the browser.",
    see1: "Hour Difference Calculator",
    see2: "Date and Time Difference Calculator",
    see3: "Future Date Calculator",
    see4: "Past Date Calculator"
  },
  pt: {
    title: "Calculadora de Diferença de Datas",
    pageTitle: "Calculadora de Diferença de Datas — Calcule Dias Entre Duas Datas | Grátis",
    meta: "Calcule a diferença entre duas datas com precisão. Obtenha o resultado exato em anos, meses e dias, mais equivalentes em semanas, dias totais, horas e minutos. Grátis, sem cadastro.",
    param1: "Data Inicial",
    param2: "Data Final",
    diff: "Diferença Exata",
    or: "Ou equivalentemente…",
    years_label: "Anos",
    months_label: "Meses",
    days_label: "Dias",
    total_months_label: "Total em Meses",
    total_weeks_label: "Total em Semanas",
    total_days_label: "Total em Dias",
    total_hours_label: "Total em Horas",
    total_minutes_label: "Total em Minutos",
    eq: "Por favor, selecione duas datas diferentes para calcular a diferença",
    features_title: "Funcionalidades",
    f_1: "Resultado exato em anos, meses e dias restantes",
    f_2: "Totais equivalentes: meses, semanas, dias, horas e minutos",
    f_3: "Tratamento automático de anos bissextos",
    f_4: "Inversão automática das datas quando informadas ao contrário",
    f_5: "Gratuita e sem cadastro",
    how_desc: "Insira uma data inicial e uma data final. A calculadora exibe instantaneamente o tempo decorrido como resultado exato (anos, meses e dias) e como totais equivalentes em semanas, dias totais, horas ou minutos conforme o intervalo. Anos bissextos e meses de diferentes durações são tratados automaticamente.",
    use_cases_title: "Principais Aplicações",
    use_1: "Gestão de Projetos: auxilia no planejamento e monitoramento de prazos, marcos e cronogramas",
    use_2: "Organização de Eventos: programe eventos com base em datas específicas e intervalos precisos",
    use_3: "Monitoramento de Prazos: mantenha-se organizado ao calcular prazos de entregas, reuniões ou compromissos",
    use_4: "Cálculos Financeiros: determine períodos de pagamento, vencimentos contratuais e investimentos",
    use_5: "Planejamento Pessoal: agende viagens, datas comemorativas e outros compromissos",
    use_6: "Educação e Estudos: calcule prazos acadêmicos, períodos letivos e cronogramas de estudos",
    use_7: "Saúde e Bem-Estar: acompanhe intervalos entre consultas, tratamentos ou ciclos de medicação",
    how_to_use_title: "Como Utilizar Esta Ferramenta",
    step_1_title: "Informe a Data Inicial",
    step_1_desc: "Clique no campo Data Inicial e selecione a primeira data no calendário ou digite diretamente.",
    step_2_title: "Informe a Data Final",
    step_2_desc: "Clique no campo Data Final e selecione a segunda data. O resultado aparece instantaneamente.",
    step_3_title: "Leia os Resultados",
    step_3_desc: "A primeira linha exibe o resultado exato em anos, meses e dias. A segunda linha mostra totais equivalentes para referência rápida.",
    ex_title: "Exemplos Comuns",
    ex1: "Planejamento de férias: de 20 de dezembro a 5 de janeiro — a calculadora mostra 16 dias, facilitando o pedido de folga ou a reserva de hospedagem.",
    ex2: "Prazo de projeto: de 1 de março a 30 de junho — o resultado é 3 meses e 29 dias, útil para planejamento de sprints e marcos.",
    ex3: "Acompanhamento de gravidez: de 14 de fevereiro a 14 de novembro — exatamente 9 meses, útil para futuros pais se planejarem.",
    ex4: "Duração de contrato: de 1 de janeiro a 31 de dezembro — 365 dias (ou 366 em ano bissexto), comum em acordos anuais.",
    faq_title: "Perguntas Frequentes",
    faq1q: "A calculadora conta o dia inicial?",
    faq1a: "Não. O cálculo conta o tempo decorrido entre as duas datas. Por exemplo, de 1 de janeiro a 3 de janeiro o resultado é 2 dias — o dia inicial não é incluído na contagem.",
    faq2q: "Esta ferramenta trata anos bissextos corretamente?",
    faq2a: "Sim. O cálculo reconhece corretamente anos bissextos. Fevereiro de 2024 tem 29 dias, e isso é refletido automaticamente nos resultados.",
    faq3q: "O que acontece se eu inserir a data final antes da data inicial?",
    faq3a: "A ferramenta inverte automaticamente as datas e calcula a diferença normalmente. Você sempre receberá um resultado positivo, independente da ordem.",
    faq4q: "Esta calculadora de diferença de datas é gratuita?",
    faq4a: "Sim, 100% gratuita. Sem cadastro, sem limites e sem taxas ocultas. Use quantas vezes precisar diretamente no navegador.",
    see1: "Calculadora de Diferença entre Horas",
    see2: "Calculadora de Diferença entre Datas e Horas",
    see3: "Calculadora de Data Futura",
    see4: "Calculadora de Data Passada"
  },
  es: {
    title: "Calculadora de Diferencia de Fechas",
    pageTitle: "Calculadora de Diferencia de Fechas — Calcular Días Entre Dos Fechas | Gratis",
    meta: "Calcula la diferencia entre dos fechas al instante. Obtén el resultado exacto en años, meses y días, más equivalentes en semanas, días totales, horas y minutos. Gratis, sin registro.",
    param1: "Fecha de Inicio",
    param2: "Fecha de Fin",
    diff: "Diferencia Exacta",
    or: "O equivalentemente…",
    years_label: "Años",
    months_label: "Meses",
    days_label: "Días",
    total_months_label: "Total en Meses",
    total_weeks_label: "Total en Semanas",
    total_days_label: "Total en Días",
    total_hours_label: "Total en Horas",
    total_minutes_label: "Total en Minutos",
    eq: "Por favor, selecciona dos fechas diferentes para calcular la diferencia",
    features_title: "Características",
    f_1: "Resultado exacto en años, meses y días restantes",
    f_2: "Totales equivalentes: meses, semanas, días, horas y minutos",
    f_3: "Manejo automático de años bisiestos",
    f_4: "Corrección automática del orden de fechas",
    f_5: "Gratis y sin registro",
    how_desc: "Introduce una fecha de inicio y una fecha de fin. La calculadora muestra al instante el tiempo transcurrido como resultado exacto (años, meses y días) y como totales equivalentes en semanas, días totales, horas o minutos según la duración.",
    use_cases_title: "Principales Aplicaciones",
    use_1: "Gestión de Proyectos: seguimiento de plazos, hitos y cronogramas",
    use_2: "Organización de Eventos: programa eventos en base a fechas específicas",
    use_3: "Control de Plazos: mantente organizado calculando fechas de entrega y citas",
    use_4: "Cálculos Financieros: determina períodos de pago y vencimientos de contratos",
    use_5: "Planificación Personal: organiza viajes, aniversarios y compromisos",
    use_6: "Educación: calcula plazos académicos y calendarios de estudio",
    use_7: "Salud: controla intervalos entre consultas y ciclos de medicación",
    how_to_use_title: "Cómo Usar Esta Herramienta",
    step_1_title: "Introduce la Fecha de Inicio",
    step_1_desc: "Haz clic en el campo Fecha de Inicio y selecciona la primera fecha.",
    step_2_title: "Introduce la Fecha de Fin",
    step_2_desc: "Haz clic en el campo Fecha de Fin y selecciona la segunda fecha. Los resultados aparecen al instante.",
    step_3_title: "Lee los Resultados",
    step_3_desc: "La primera fila muestra el desglose exacto en años, meses y días. La segunda fila muestra totales equivalentes.",
    ex_title: "Ejemplos Comunes",
    ex1: "Planificación de vacaciones: del 20 de diciembre al 5 de enero — la calculadora muestra 16 días.",
    ex2: "Plazo de proyecto: del 1 de marzo al 30 de junio — el resultado es 3 meses y 29 días.",
    ex3: "Seguimiento de embarazo: del 14 de febrero al 14 de noviembre — exactamente 9 meses.",
    ex4: "Duración de contrato: del 1 de enero al 31 de diciembre — 365 días (o 366 en año bisiesto).",
    faq_title: "Preguntas Frecuentes",
    faq1q: "¿La calculadora cuenta el día de inicio?",
    faq1a: "No. El cálculo cuenta el tiempo transcurrido entre las dos fechas. Por ejemplo, del 1 al 3 de enero el resultado es 2 días.",
    faq2q: "¿Esta herramienta maneja correctamente los años bisiestos?",
    faq2a: "Sí. El cálculo reconoce correctamente los años bisiestos. Febrero de 2024 tiene 29 días y esto se refleja automáticamente.",
    faq3q: "¿Qué pasa si introduzco la fecha de fin antes que la de inicio?",
    faq3a: "La herramienta invierte automáticamente las fechas y calcula la diferencia normalmente. Siempre obtendrás un resultado positivo.",
    faq4q: "¿Esta calculadora es gratuita?",
    faq4a: "Sí, 100% gratuita. Sin registro, sin límites y sin tarifas ocultas.",
    see1: "Calculadora de Diferencia de Horas",
    see2: "Calculadora de Diferencia de Fecha y Hora",
    see3: "Calculadora de Fecha Futura",
    see4: "Calculadora de Fecha Pasada"
  },
  fr: {
    title: "Calculateur de Différence entre Dates",
    pageTitle: "Calculateur de Différence entre Dates — Calculer les Jours entre Deux Dates | Gratuit",
    meta: "Calculez la différence entre deux dates instantanément. Obtenez le résultat exact en années, mois et jours, plus les équivalents en semaines, jours totaux, heures et minutes. Gratuit, sans inscription.",
    param1: "Date de Début",
    param2: "Date de Fin",
    diff: "Différence Exacte",
    or: "Ou de manière équivalente…",
    years_label: "Années",
    months_label: "Mois",
    days_label: "Jours",
    total_months_label: "Total en Mois",
    total_weeks_label: "Total en Semaines",
    total_days_label: "Total en Jours",
    total_hours_label: "Total en Heures",
    total_minutes_label: "Total en Minutes",
    eq: "Veuillez sélectionner deux dates différentes pour calculer la différence",
    features_title: "Fonctionnalités",
    f_1: "Résultat exact en années, mois et jours restants",
    f_2: "Totaux équivalents : mois, semaines, jours, heures et minutes",
    f_3: "Gestion automatique des années bissextiles",
    f_4: "Correction automatique de l'ordre des dates",
    f_5: "Gratuit et sans inscription",
    how_desc: "Entrez une date de début et une date de fin. Le calculateur affiche instantanément le temps écoulé sous forme de résultat exact (années, mois et jours) et de totaux équivalents en semaines, jours totaux, heures ou minutes selon la durée.",
    use_cases_title: "Principales Applications",
    use_1: "Gestion de Projet : suivi des délais, jalons et plannings",
    use_2: "Organisation d'Événements : planifiez des événements sur des dates précises",
    use_3: "Suivi des Délais : restez organisé en calculant les dates de livraison et rendez-vous",
    use_4: "Calculs Financiers : déterminez les périodes de paiement et échéances contractuelles",
    use_5: "Planification Personnelle : organisez voyages, anniversaires et engagements",
    use_6: "Éducation : calculez les délais académiques et calendriers scolaires",
    use_7: "Santé : suivez les intervalles entre consultations et cycles de médication",
    how_to_use_title: "Comment Utiliser Cet Outil",
    step_1_title: "Entrez la Date de Début",
    step_1_desc: "Cliquez sur le champ Date de Début et sélectionnez la première date.",
    step_2_title: "Entrez la Date de Fin",
    step_2_desc: "Cliquez sur le champ Date de Fin et sélectionnez la deuxième date. Les résultats apparaissent instantanément.",
    step_3_title: "Lisez les Résultats",
    step_3_desc: "La première ligne affiche la décomposition exacte en années, mois et jours. La deuxième ligne montre les totaux équivalents.",
    ex_title: "Exemples Courants",
    ex1: "Planification de vacances : du 20 décembre au 5 janvier — le calculateur affiche 16 jours.",
    ex2: "Délai de projet : du 1er mars au 30 juin — le résultat est 3 mois et 29 jours.",
    ex3: "Suivi de grossesse : du 14 février au 14 novembre — exactement 9 mois.",
    ex4: "Durée de contrat : du 1er janvier au 31 décembre — 365 jours (ou 366 en année bissextile).",
    faq_title: "Questions Fréquentes",
    faq1q: "Le calculateur compte-t-il le jour de début ?",
    faq1a: "Non. Le calcul compte le temps écoulé entre les deux dates. Par exemple, du 1er au 3 janvier le résultat est 2 jours.",
    faq2q: "Cet outil gère-t-il correctement les années bissextiles ?",
    faq2a: "Oui. Le calcul reconnaît correctement les années bissextiles. Février 2024 a 29 jours, ce qui est automatiquement reflété.",
    faq3q: "Que se passe-t-il si j'entre la date de fin avant la date de début ?",
    faq3a: "L'outil inverse automatiquement les dates et calcule la différence normalement. Vous obtiendrez toujours un résultat positif.",
    faq4q: "Ce calculateur est-il gratuit ?",
    faq4a: "Oui, 100% gratuit. Sans inscription, sans limites et sans frais cachés.",
    see1: "Calculateur de Différence d'Heures",
    see2: "Calculateur de Différence de Date et Heure",
    see3: "Calculateur de Date Future",
    see4: "Calculateur de Date Passée"
  },
  it: {
    title: "Calcolatore di Differenza tra Date",
    pageTitle: "Calcolatore di Differenza tra Date — Calcola i Giorni tra Due Date | Gratis",
    meta: "Calcola la differenza tra due date all'istante. Ottieni il risultato esatto in anni, mesi e giorni, più equivalenti in settimane, giorni totali, ore e minuti. Gratis, senza registrazione.",
    param1: "Data di Inizio",
    param2: "Data di Fine",
    diff: "Differenza Esatta",
    or: "Oppure equivalentemente…",
    years_label: "Anni",
    months_label: "Mesi",
    days_label: "Giorni",
    total_months_label: "Totale Mesi",
    total_weeks_label: "Totale Settimane",
    total_days_label: "Totale Giorni",
    total_hours_label: "Totale Ore",
    total_minutes_label: "Totale Minuti",
    eq: "Seleziona due date diverse per calcolare la differenza",
    features_title: "Funzionalità",
    f_1: "Risultato esatto in anni, mesi e giorni rimanenti",
    f_2: "Totali equivalenti: mesi, settimane, giorni, ore e minuti",
    f_3: "Gestione automatica degli anni bisestili",
    f_4: "Correzione automatica dell'ordine delle date",
    f_5: "Gratuito e senza registrazione",
    how_desc: "Inserisci una data di inizio e una data di fine. Il calcolatore mostra istantaneamente il tempo trascorso come risultato esatto (anni, mesi e giorni) e come totali equivalenti in settimane, giorni totali, ore o minuti a seconda della durata.",
    use_cases_title: "Principali Applicazioni",
    use_1: "Gestione di Progetti: monitoraggio di scadenze, traguardi e cronoprogrammi",
    use_2: "Organizzazione di Eventi: pianifica eventi basandoti su date precise",
    use_3: "Controllo delle Scadenze: rimani organizzato calcolando consegne e appuntamenti",
    use_4: "Calcoli Finanziari: determina periodi di pagamento e scadenze contrattuali",
    use_5: "Pianificazione Personale: organizza viaggi, anniversari e impegni",
    use_6: "Istruzione: calcola scadenze accademiche e calendari scolastici",
    use_7: "Salute: monitora intervalli tra visite e cicli di medicazione",
    how_to_use_title: "Come Usare Questo Strumento",
    step_1_title: "Inserisci la Data di Inizio",
    step_1_desc: "Clicca sul campo Data di Inizio e seleziona la prima data.",
    step_2_title: "Inserisci la Data di Fine",
    step_2_desc: "Clicca sul campo Data di Fine e seleziona la seconda data. I risultati appaiono istantaneamente.",
    step_3_title: "Leggi i Risultati",
    step_3_desc: "La prima riga mostra il dettaglio esatto in anni, mesi e giorni. La seconda riga mostra i totali equivalenti.",
    ex_title: "Esempi Comuni",
    ex1: "Pianificazione vacanze: dal 20 dicembre al 5 gennaio — il calcolatore mostra 16 giorni.",
    ex2: "Scadenza progetto: dal 1° marzo al 30 giugno — il risultato è 3 mesi e 29 giorni.",
    ex3: "Monitoraggio gravidanza: dal 14 febbraio al 14 novembre — esattamente 9 mesi.",
    ex4: "Durata contratto: dal 1° gennaio al 31 dicembre — 365 giorni (o 366 in anno bisestile).",
    faq_title: "Domande Frequenti",
    faq1q: "Il calcolatore conta il giorno di inizio?",
    faq1a: "No. Il calcolo conta il tempo trascorso tra le due date. Ad esempio, dal 1° al 3 gennaio il risultato è 2 giorni.",
    faq2q: "Questo strumento gestisce correttamente gli anni bisestili?",
    faq2a: "Sì. Il calcolo riconosce correttamente gli anni bisestili. Febbraio 2024 ha 29 giorni e questo viene riflesso automaticamente.",
    faq3q: "Cosa succede se inserisco la data di fine prima di quella di inizio?",
    faq3a: "Lo strumento inverte automaticamente le date e calcola la differenza normalmente. Otterrai sempre un risultato positivo.",
    faq4q: "Questo calcolatore è gratuito?",
    faq4a: "Sì, 100% gratuito. Senza registrazione, senza limiti e senza costi nascosti.",
    see1: "Calcolatore di Differenza tra Ore",
    see2: "Calcolatore di Differenza tra Data e Ora",
    see3: "Calcolatore di Data Futura",
    see4: "Calcolatore di Data Passata"
  },
  id: {
    title: "Kalkulator Selisih Tanggal",
    pageTitle: "Kalkulator Selisih Tanggal — Hitung Hari antara Dua Tanggal | Gratis",
    meta: "Hitung selisih antara dua tanggal secara instan. Dapatkan hasil tepat dalam tahun, bulan, dan hari, plus setara dalam minggu, total hari, jam, dan menit. Gratis, tanpa pendaftaran.",
    param1: "Tanggal Mulai",
    param2: "Tanggal Akhir",
    diff: "Selisih Tepat",
    or: "Atau setara dengan…",
    years_label: "Tahun",
    months_label: "Bulan",
    days_label: "Hari",
    total_months_label: "Total Bulan",
    total_weeks_label: "Total Minggu",
    total_days_label: "Total Hari",
    total_hours_label: "Total Jam",
    total_minutes_label: "Total Menit",
    eq: "Pilih dua tanggal yang berbeda untuk menghitung selisihnya",
    features_title: "Fitur",
    f_1: "Hasil tepat dalam tahun, bulan, dan hari yang tersisa",
    f_2: "Total setara: bulan, minggu, hari, jam, dan menit",
    f_3: "Penanganan tahun kabisat secara otomatis",
    f_4: "Koreksi urutan tanggal secara otomatis",
    f_5: "Gratis dan tanpa pendaftaran",
    how_desc: "Masukkan tanggal mulai dan tanggal akhir. Kalkulator langsung menampilkan waktu yang berlalu sebagai hasil tepat (tahun, bulan, dan hari) dan total setara dalam minggu, total hari, jam, atau menit tergantung durasinya.",
    use_cases_title: "Kegunaan Utama",
    use_1: "Manajemen Proyek: pantau tenggat waktu, tonggak, dan jadwal",
    use_2: "Perencanaan Acara: jadwalkan acara berdasarkan tanggal dan interval yang tepat",
    use_3: "Pemantauan Tenggat: tetap terorganisir dengan menghitung batas waktu pengiriman dan janji",
    use_4: "Perhitungan Keuangan: tentukan periode pembayaran dan jatuh tempo kontrak",
    use_5: "Perencanaan Pribadi: atur perjalanan, hari jadi, dan komitmen lainnya",
    use_6: "Pendidikan: hitung tenggat akademik dan jadwal belajar",
    use_7: "Kesehatan: pantau interval antara konsultasi dan siklus pengobatan",
    how_to_use_title: "Cara Menggunakan Alat Ini",
    step_1_title: "Masukkan Tanggal Mulai",
    step_1_desc: "Klik kolom Tanggal Mulai dan pilih tanggal pertama.",
    step_2_title: "Masukkan Tanggal Akhir",
    step_2_desc: "Klik kolom Tanggal Akhir dan pilih tanggal kedua. Hasilnya langsung muncul.",
    step_3_title: "Baca Hasilnya",
    step_3_desc: "Baris pertama menampilkan rincian tepat dalam tahun, bulan, dan hari. Baris kedua menampilkan total setara.",
    ex_title: "Contoh Umum",
    ex1: "Perencanaan liburan: dari 20 Desember hingga 5 Januari — kalkulator menampilkan 16 hari.",
    ex2: "Tenggat proyek: dari 1 Maret hingga 30 Juni — hasilnya adalah 3 bulan 29 hari.",
    ex3: "Pemantauan kehamilan: dari 14 Februari hingga 14 November — tepat 9 bulan.",
    ex4: "Durasi kontrak: dari 1 Januari hingga 31 Desember — 365 hari (atau 366 di tahun kabisat).",
    faq_title: "Pertanyaan Umum",
    faq1q: "Apakah kalkulator menghitung tanggal mulai?",
    faq1a: "Tidak. Perhitungan menghitung waktu yang berlalu di antara dua tanggal. Misalnya, dari 1 Januari hingga 3 Januari hasilnya adalah 2 hari.",
    faq2q: "Apakah alat ini menangani tahun kabisat dengan benar?",
    faq2a: "Ya. Perhitungan mengenali tahun kabisat dengan benar. Februari 2024 memiliki 29 hari dan ini secara otomatis tercermin dalam hasilnya.",
    faq3q: "Apa yang terjadi jika saya memasukkan tanggal akhir sebelum tanggal mulai?",
    faq3a: "Alat ini secara otomatis membalik tanggal dan menghitung selisih seperti biasa. Anda selalu mendapatkan hasil positif.",
    faq4q: "Apakah kalkulator ini gratis?",
    faq4a: "Ya, 100% gratis. Tanpa pendaftaran, tanpa batasan, dan tanpa biaya tersembunyi.",
    see1: "Selisih Waktu",
    see2: "Selisih Tanggal dan Waktu",
    see3: "Kalkulator Tanggal Masa Depan",
    see4: "Kalkulator Tanggal Masa Lalu"
  }
}
</i18n>
