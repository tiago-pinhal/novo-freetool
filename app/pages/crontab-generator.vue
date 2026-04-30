<script setup lang="ts">
const { t, locale } = useI18n()
const lang = computed(() => locale.value !== 'pt' ? locale.value : 'pt_BR')

const state = reactive({
  cron: '* * * * * *',
  result: '',
  pos: 0,
  tab: 'edition',
  hasSeconds: true,
  error: false,
  showAds: false,
})

const stateDates = reactive({
  currDate: null as Date | null,
  nexts: [] as Date[],
})

const fieldDesc = computed(() => t('field').split(','))
const faqs = computed(() =>
  Array.from({ length: 6 }, (_, i) => ({ question: t(`faq${i + 1}q`), answer: t(`faq${i + 1}a`) }))
)

const allowedValues = [
  '0 ... 59', // Sec
  '0 ... 59', // Min
  '0 ... 23', // Hour
  '1 ... 31 ● L', // Day of Month
  '1 ... 12 ● JAN ... DEC', // Month
  '0 ... 6 ● SUN ... SAT ● L ● #', // Day of Week
]

const cronParts = computed(() => {
  const parts = state.cron.trim().split(/\s+/)
  return parts
})

let lastGeneratedDate: Date

onMounted(() => {
  loadCurrDateTime()
  generate()
})

watch(() => state.cron, generate)

function position(value: number) {
  if (state.cron.trim() === '') {
    state.pos = -1
    return
  }
  const count = state.cron.substring(0, value).split(' ').length
  const spaces = state.cron.split(' ').length
  state.hasSeconds = spaces > 5
  state.pos = spaces <= 5 ? count : count - 1
}

function generate() {
  state.error = false
  stateDates.nexts = []
  
  const valid = isValidCron(state.cron)
  if (valid && stateDates.currDate) {
    stateDates.nexts = getNexts(stateDates.currDate, 5)
  } else if (!valid) {
    state.result = ''
    state.error = true
  }
}

function isValidCron(cron: string) {
  if (!(window as any).cronstrue) return false
  try {
    state.result = (window as any).cronstrue.toString(cron, { locale: lang.value })
    return true
  } catch (error: any) {
    return false
  }
}

function getNexts(currDate: Date, qty: number): Date[] {
  if (!(window as any).Cron) return []
  try {
    const aux = new (window as any).Cron(state.cron).nextRuns(qty, currDate)
    lastGeneratedDate = aux[qty - 1]
    return aux
  } catch {
    return []
  }
}

function getMoreDates() {
  stateDates.nexts.push(...getNexts(lastGeneratedDate, 3))
}

function loadCurrDateTime() {
  const date = new Date()
  stateDates.currDate = date

  // Sincroniza o valor do input datetime-local quando ele estiver disponível
  setTimeout(() => {
    const el = document.getElementById('dt') as HTMLInputElement
    if (el) {
      const localDate = new Date(date.getTime() - date.getTimezoneOffset() * 60000)
      el.value = localDate.toISOString().slice(0, 16)
    }
  }, 200)
}

function onDateChange(e: Event) {
  stateDates.currDate = new Date((e.target as HTMLInputElement).value)
  generate()
}

function formatDt(dt: Date): string {
  const fmt: Intl.DateTimeFormatOptions = {
    year: 'numeric', month: '2-digit', day: '2-digit', hour: '2-digit', minute: '2-digit',
  }
  if (state.hasSeconds) fmt.second = '2-digit'
  return dt ? dt.toLocaleTimeString([], fmt) : ''
}

usePageJsonLd({
  name: t('title'),
  description: t('meta'),
  features: Array.from({ length: 6 }, (_, i) => t(`feat${i + 1}`)),
  faq: Array.from({ length: 6 }, (_, i) => ({ question: t(`faq${i + 1}q`), answer: t(`faq${i + 1}a`) })),
  howToName: t('t_how'),
  howToSteps: [
    { name: t('how1_t'), text: t('how1') },
    { name: t('how2_t'), text: t('how2') },
    { name: t('how3_t'), text: t('how3') },
    { name: t('how4_t'), text: t('how4') },
  ],
})

useHead({
  title: t('pageTitle'),
  meta: [{ name: 'description', content: t('meta') }],
  script: [
    { src: "https://unpkg.com/cronstrue@latest/dist/cronstrue.min.js", defer: true, crossorigin: "anonymous", referrerpolicy: "no-referrer", onload: generate },
    { src: "https://cdn.jsdelivr.net/npm/croner@10.0.1/dist/croner.umd.min.js", defer: true, crossorigin: "anonymous", referrerpolicy: "no-referrer", onload: generate },
    { src: computed(() => `https://unpkg.com/cronstrue@latest/locales/${lang.value}.min.js`), defer: true, crossorigin: "anonymous", referrerpolicy: "no-referrer", onload: generate },
  ]
})

defineI18nRoute({
  paths: {
    pt: '/gerador-de-crontab',
    es: '/generador-de-crontab',
    fr: '/generateur-de-crontab',
    it: '/generatore-di-crontab',
    id: '/generator-crontab',
  },
})
</script>

<template>
  <ToolPage
    :title="t('title')"
    :description="t('meta')"
    :show-ads="state.showAds"
    :see-also-links="[
      { label: t('see1'), to: 'uuid-generator' },
      { label: t('see2'), to: 'fancy-letters' },
      { label: t('see3'), to: 'credit-card-generator' },
      { label: t('see4'), to: 'birthday-generator' },
    ]"
  >
    <ToolTabs
      :active-tab="state.tab"
      @update:active-tab="(val) => (state.tab = val)"
      :tabs="[
        { id: 'edition', label: t('edition') },
        { id: 'results', label: t('results') },
      ]"
    >
      <!-- Tab: Editor -->
      <template #edition>
        <div class="animate-in fade-in slide-in-from-bottom-4 duration-500">
          <!-- Description + Input Group -->
          <div class="flex flex-col gap-1.5">
            <!-- Result / error description -->
            <div
              v-if="state.cron"
              class="min-h-[1.5rem] px-1 text-sm font-bold transition-colors"
              :class="state.error ? 'text-error' : 'text-primary'"
            >
              {{ state.error ? t('error') : state.result }}
            </div>

            <!-- Cron input -->
            <div class="relative group">
            <input
              id="cron"
              v-model="state.cron"
              type="text"
              placeholder="* * * * * *"
              autofocus
              autocomplete="off"
              class="input input-lg input-bordered w-full h-20 text-3xl sm:text-4xl font-mono pr-16 bg-base-200/40 focus:bg-base-100 focus:border-primary focus:ring-4 focus:ring-primary/10 transition-all shadow-inner tracking-widest"
              @keyup="position(($event.target as HTMLInputElement).selectionStart ?? 0); state.showAds = true"
              @mouseup="position(($event.target as HTMLInputElement).selectionStart ?? 0)"
            />
            <button
              @click.prevent="state.cron = '* * * * * *'"
              class="btn btn-ghost btn-circle btn-sm absolute right-3 top-1/2 -translate-y-1/2 hover:bg-primary/20 text-base-content/50 hover:text-primary transition-all"
              :aria-label="t('reset')"
            >
              <svg xmlns="http://www.w3.org/2000/svg" class="w-6 h-6" viewBox="0 0 24 24" aria-hidden="true">
                <path fill="currentColor" d="M2 12C2 16.97 6.03 21 11 21C13.39 21 15.68 20.06 17.4 18.4L15.9 16.9C14.63 18.25 12.86 19 11 19C4.76 19 1.64 11.46 6.05 7.05C10.46 2.64 18 5.77 18 12H15L19 16H19.1L23 12H20C20 7.03 15.97 3 11 3C6.03 3 2 7.03 2 12Z" />
              </svg>
            </button>
          </div>
        </div>

          <!-- Field Stats (Unified DaisyUI Style) -->
          <div v-if="state.cron" class="stats shadow w-full bg-base-200/50 border border-base-content/10 mt-6 stats-vertical lg:stats-horizontal overflow-hidden">
            <div
              v-for="idx in (state.hasSeconds ? [0, 1, 2, 3, 4, 5] : [1, 2, 3, 4, 5])"
              :key="idx"
              class="stat p-3 sm:p-4 place-items-center transition-all duration-300 border-base-content/5"
              :class="state.pos === idx ? 'bg-primary text-primary-content !opacity-100' : ''"
            >
              <div class="stat-title text-[11px] uppercase font-black tracking-widest" :class="state.pos === idx ? 'text-primary-content' : 'text-base-content/80'">
                {{ fieldDesc[idx] }}
              </div>
              <div class="stat-value text-xl sm:text-2xl font-mono font-black">
                {{ cronParts[state.hasSeconds ? idx : idx - 1] || '*' }}
              </div>
              <div class="stat-desc text-xs font-bold" :class="state.pos === idx ? 'text-primary-content' : 'text-base-content/70'">
                {{ allowedValues[idx] }}
              </div>
            </div>
          </div>

          <!-- Legends -->
          <div v-if="state.cron">
            <p class="text-base uppercase tracking-widest text-base-content/50 mt-4 mb-2">{{ t('legends') }}</p>
            <div class="overflow-x-auto rounded-box border border-base-content/5 bg-base-100/50 mt-2">
              <table class="table table-sm">
                <tbody>
                  <tr class="border-base-content/5"><th class="font-mono text-primary w-20 text-lg">*</th><td class="text-base-content/80 font-medium">{{ t('all') }}</td></tr>
                  <tr class="border-base-content/5"><th class="font-mono text-primary text-lg">X,Y</th><td class="text-base-content/80 font-medium">{{ t('in') }}</td></tr>
                  <tr class="border-base-content/5"><th class="font-mono text-primary text-lg">X-Y</th><td class="text-base-content/80 font-medium">{{ t('from') }}</td></tr>
                  <tr class="border-base-content/5"><th class="font-mono text-primary text-lg">*/X</th><td class="text-base-content/80 font-medium">{{ t('each') }}</td></tr>
                  <tr class="border-base-content/5"><th class="font-mono text-primary text-lg">Y/X</th><td class="text-base-content/80 font-medium">{{ t('eachFrom') }}</td></tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </template>

      <!-- Tab: Results -->
      <template #results>
        <div class="space-y-5 animate-in fade-in slide-in-from-bottom-4 duration-500">
          <!-- Reference date -->
          <div class="flex flex-col gap-1">
            <label for="dt" class="label-text font-bold text-base-content/80 px-1">{{ t('dt') }}</label>
            <input
              id="dt"
              type="datetime-local"
              class="input input-bordered w-full sm:w-64 bg-base-100 focus:bg-base-100 transition-all"
              @change="onDateChange"
            />
          </div>

          <!-- Next scheduled dates -->
          <div v-if="stateDates.nexts.length" class="flex flex-col gap-1 w-fit">
            <div
              v-for="(dt, i) in stateDates.nexts"
              :key="i"
              class="px-4 py-2 rounded-lg font-mono text-sm border border-base-content/5 bg-base-200/40 w-fit transition-all hover:bg-base-200"
            >
              {{ formatDt(dt) }}
            </div>
          </div>

          <!-- Load more -->
          <button
            v-if="stateDates.nexts.length"
            type="button"
            @click.prevent="getMoreDates"
            class="btn btn-primary btn-sm w-fit"
          >
            <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4 mr-1" viewBox="0 0 24 24"><path fill="currentColor" d="M19 13h-6v6h-2v-6H5v-2h6V5h2v6h6v2z"/></svg>
            {{ t('add') }}
          </button>
        </div>
      </template>
    </ToolTabs>

    <!-- Info section -->
    <template #info>
      <div class="space-y-8">
        <section>
          <p class="text-base-content/80 leading-relaxed">{{ t('d1') }}.</p>
        </section>

        <FeatureSection
          :title="t('t5')"
          :items="[t('d51'), t('d52'), t('d53'), t('d54'), t('d55'), t('d56')]"
        />

        <UseCaseSection
          :title="t('t6')"
          :description="t('d6')"
          :items="[
            { title: t('d61_t'), description: t('d61_d') },
            { title: t('d62_t'), description: t('d62_d') },
            { title: t('d63_t'), description: t('d63_d') },
            { title: t('d64_t'), description: t('d64_d') },
          ]"
        />

        <!-- Special characters -->
        <section>
          <h2 class="text-2xl font-bold text-base-content mb-4 flex items-center gap-2">
            <Icon name="heroicons:code-bracket-20-solid" class="text-primary w-6 h-6" aria-hidden="true" />
            {{ t('t7') }}
          </h2>
          <p class="text-base-content/70 text-sm mb-4">{{ t('d7') }}</p>
          <div class="overflow-x-auto">
            <table class="table table-sm w-fit border border-base-content/10 rounded-xl text-sm">
              <tbody>
                <tr v-for="i in 4" :key="i">
                  <th class="font-mono text-primary w-20">{{ ['*', 'X,Y', 'X-Y', '*/X'][i - 1] }}</th>
                  <td class="text-base-content/80">{{ t(`d7${i}`) }}</td>
                </tr>
              </tbody>
            </table>
          </div>
        </section>

        <HowToSection
          :title="t('t_how')"
          :items="[
            { title: t('how1_t'), description: t('how1') },
            { title: t('how2_t'), description: t('how2') },
            { title: t('how3_t'), description: t('how3') },
            { title: t('how4_t'), description: t('how4') },
          ]"
        />

        <!-- Real-world examples -->
        <section>
          <h2 class="text-2xl font-bold text-base-content mb-4 flex items-center gap-2">
            <Icon name="heroicons:light-bulb-20-solid" class="text-primary w-6 h-6" aria-hidden="true" />
            {{ t('t_real') }}
          </h2>
          <p class="text-sm text-base-content/60 mb-3">{{ t('real_intro') }}</p>
          <ul class="space-y-2">
            <li v-for="i in 5" :key="i" class="flex gap-3 font-mono text-sm text-base-content/80">
              <span class="text-primary mt-0.5 shrink-0">•</span>
              <span>{{ t(`real${i}`) }}</span>
            </li>
          </ul>
        </section>

        <!-- Edge cases -->
        <section>
          <h2 class="text-2xl font-bold text-base-content mb-4 flex items-center gap-2">
            <Icon name="heroicons:exclamation-triangle-20-solid" class="text-primary w-6 h-6" aria-hidden="true" />
            {{ t('t_edge') }}
          </h2>
          <p class="text-sm text-base-content/60 mb-3">{{ t('edge_intro') }}</p>
          <ul class="space-y-2">
            <li v-for="i in 3" :key="i" class="flex gap-3 text-sm text-base-content/80">
              <span class="text-primary mt-0.5 shrink-0">•</span>
              <span>{{ t(`edge${i}`) }}</span>
            </li>
          </ul>
        </section>

        <FaqSection :title="t('t_faq')" :items="faqs" />
      </div>
    </template>
  </ToolPage>
</template>

<i18n lang="yaml">
{
  en: {
    title: "Crontab Generator",
    pageTitle: "Crontab Generator — 5 and 6 Field CRON Expressions",
    meta: "Free online crontab generator. Create and validate CRON expressions with 5 or 6 fields instantly — see next scheduled dates from any reference date.",
    d1: "With it, you can generate Crontab expressions with 5 or 6 fields easily and also view scheduling dates based on the generated expression",
    edition: "Editor",
    results: "Results",
    error: "Enter a valid Cron expression",
    values: "Allowed values",
    optional: "optional",
    all: "Every value in the field",
    in: "In X and Y",
    from: "From X to Y",
    each: "Every X",
    eachFrom: "Every X, starting from Y",
    legends: "Legend",
    dt: "Start simulation from",
    add: "Load more results",
    reset: "Reset",
    field: "Second,Minute,Hour,Day,Month,Weekday",
    see1: "UUID Generator",
    see2: "Fancy Letters",
    see3: "Credit Card Generator",
    see4: "Birthday Generator",
    t5: "What Is a CRON Expression?",
    d51: "Second (0–59): defines the second at which the task runs",
    d52: "Minute (0–59): defines the minute at which the task runs",
    d53: "Hour (0–23): defines the hour of the day at which the task runs",
    d54: "Day of month (1–31): defines the day of the month",
    d55: "Month (1–12 or JAN–DEC): defines the month",
    d56: "Day of week (0–7 or SUN–SAT): defines the weekday (0 and 7 represent Sunday)",
    t6: "CRON Expression Examples",
    d6: "Common scheduling scenarios and their CRON expressions",
    d61_t: "* * * * *",
    d61_d: "Runs every minute",
    d62_t: "0 0 * * *",
    d62_d: "Runs at midnight every day",
    d63_t: "0 12 * * MON-FRI",
    d63_d: "Runs at noon from Monday to Friday",
    d64_t: "0 0 1 * *",
    d64_d: "Runs at midnight on the first day of every month",
    t7: "Special Characters",
    d7: "Understanding the special characters available in CRON expressions",
    d71: "Asterisk (*): represents all possible values for that field. For example, an asterisk in the minutes field means 'every minute'",
    d72: "Comma (,): allows listing multiple values. For example, '1,15' in the hours field means 'at 1:00 and 15:00'",
    d73: "Hyphen (-): defines a range. For example, '1-5' in the weekday field means 'Monday through Friday'",
    d74: "Slash (/): defines step values. For example, '*/2' in the minutes field means 'every 2 minutes'",
    t_how: "How It Works",
    how1_t: "Type your expression",
    how1: "Type or paste any CRON expression — the tool validates and describes it in real time as you type",
    how2_t: "Read the description",
    how2: "The tool translates the expression into plain language so you can confirm its meaning without memorizing the syntax",
    how3_t: "Preview next runs",
    how3: "Switch to the Results tab to see upcoming scheduled dates from any reference date",
    how4_t: "Load more dates",
    how4: "Click 'Load more results' to fetch additional future dates and verify long-term schedule behavior",
    t_real: "Real-World Examples",
    real_intro: "Common scheduling scenarios and the exact CRON expressions that power them",
    real1: "0 2 * * * — Daily database backup at 2 AM",
    real2: "*/15 * * * * — Health check every 15 minutes",
    real3: "0 9 * * MON — Weekly report email every Monday at 9 AM",
    real4: "0 0 1 1 * — Annual archiving job on January 1st at midnight",
    real5: "0 18 * * FRI — Weekend summary every Friday at 6 PM",
    t_edge: "Edge Cases and Advanced Patterns",
    edge_intro: "Tricky scheduling scenarios that most tools don't explain",
    edge1: "0,30 * * * * — Runs at minutes 0 and 30 of every hour, i.e. twice per hour",
    edge2: "30 0 2 * * * — 6-field expression: runs exactly at 02:00:30 (second-level precision)",
    edge3: "0 8-18 * * 1-5 — Runs every hour from 8 AM to 6 PM, only on weekdays",
    t_faq: "Frequently Asked Questions",
    faq1q: "What is a crontab expression?",
    faq1a: "A crontab expression is a string of 5 or 6 space-separated fields that defines when a scheduled task (cron job) should run on Unix systems. Each field represents a time unit: minute, hour, day of month, month and day of week — with an optional seconds field at the start.",
    faq2q: "What is the difference between 5-field and 6-field CRON expressions?",
    faq2a: "A 5-field expression starts at the minute level (minute hour day month weekday), while a 6-field expression adds a seconds field at the start (second minute hour day month weekday). The 6-field format is used by schedulers like Quartz (Java) and some modern task runners.",
    faq3q: "How do I run a task every 5 minutes?",
    faq3a: "Use the expression */5 * * * * which means 'every 5 minutes'. The */5 syntax means 'every step of 5 starting from 0', so the task runs at minutes 0, 5, 10, 15, and so on.",
    faq4q: "What does */2 mean in a CRON expression?",
    faq4a: "The */2 notation means 'every 2 units'. In the minutes field it means every 2 minutes (0, 2, 4, …). In the hours field it means every 2 hours (0, 2, 4, …). The slash / defines a step value over the full range.",
    faq5q: "Can I use month and weekday names instead of numbers?",
    faq5a: "Yes. You can use JAN through DEC for months and SUN through SAT (or MON-FRI) for weekdays. Names are case-insensitive on most systems. For example, 0 9 * * MON-FRI runs at 9 AM on all weekdays.",
    faq6q: "Is there a way to preview when my cron job will run?",
    faq6a: "Yes — switch to the Results tab in this tool. It calculates and lists the upcoming run dates based on your expression and a reference start date. You can also change the reference date to simulate scheduling from any point in time.",
    feat1: "Supports 5 and 6 field CRON expressions",
    feat2: "Real-time validation and natural language description",
    feat3: "Preview next scheduled dates from any reference date",
    feat4: "Second-level scheduling precision",
    feat5: "Support for L, # special characters and ranges",
    feat6: "Available in 6 languages"
  },
  pt: {
    title: "Gerador de Crontab",
    pageTitle: "Gerador de Crontab com 5 ou 6 campos",
    meta: "Nossa ferramenta de edição de expressões Crontab simplifica a criação e compreensão de expressões CRON",
    d1: "Com ela, você poderá gerar expressões Crontab com 5 ou 6 campos com facilidade e também visualizar as datas de agendamento com base na expressão gerada",
    edition: "Edição",
    results: "Resultados",
    error: "Informe uma Expressão Cron válida",
    values: "Valores Possíveis",
    optional: "Opcional",
    all: "Para cada valor do campo",
    in: "Em X e Y",
    from: "De X a Y",
    each: "A cada X",
    eachFrom: "A cada X, a partir de Y",
    legends: "Legendas",
    dt: "Iniciar simulação com",
    add: "Adicionar mais resultados",
    reset: "Reiniciar",
    field: "Segundo,Minuto,Hora,Dia,Mês,Semana",
    see1: "Gerador de UUID",
    see2: "Letras Diferentes",
    see3: "Gerador de Cartão de Crédito",
    see4: "Gerador de Data de Nascimento",
    t5: "O Que é uma Expressão CRON?",
    d51: "Segundo (0-59): define o segundo em que a tarefa será executada",
    d52: "Minuto (0-59): define o minuto em que a tarefa será executada",
    d53: "Hora (0-23): define a hora do dia em que a tarefa será executada",
    d54: "Dia do mês (1-31): define o dia do mês em que a tarefa será executada",
    d55: "Mês (1-12 ou JAN-DEC): define o mês em que a tarefa será executada",
    d56: "Dia da semana (0-7 ou SUN-SAT): define o dia da semana (0 e 7 representam domingo)",
    t6: "Exemplos de Expressões CRON",
    d6: "Alguns exemplos de expressões CRON e seus significados",
    d61_t: "* * * * *",
    d61_d: "A tarefa é executada a cada minuto",
    d62_t: "0 0 * * *",
    d62_d: "Executa a tarefa à meia-noite todos os dias",
    d63_t: "0 12 * * MON-FRI",
    d63_d: "Executa a tarefa ao meio-dia de segunda a sexta-feira",
    d64_t: "0 0 1 * *",
    d64_d: "A tarefa será executada à meia-noite no primeiro dia de cada mês",
    t7: "Caracteres Especiais",
    d7: "Entenda o significado dos caracteres especiais que podem ser utilizados na expressão CRON",
    d71: "Asterisco (*): representa todos os valores possíveis para aquele campo. Por exemplo, um asterisco no campo de minutos significa 'a cada minuto'",
    d72: "Vírgula (,): permite listar valores múltiplos. Por exemplo, '1,15' no campo de horas significa 'à 1h e às 15h'",
    d73: "Hífen (-): define um intervalo de valores. Por exemplo, '1-5' no campo de dias da semana significa 'de segunda a sexta-feira'",
    d74: "Barra (/): define incrementos. Por exemplo, '*/2' no campo de minutos significa 'a cada 2 minutos'",
    t_how: "Como Funciona",
    how1_t: "Digite ou cole a expressão",
    how1: "Digite ou cole qualquer expressão CRON no campo de entrada — a ferramenta valida instantaneamente enquanto você digita",
    how2_t: "Leia a descrição",
    how2: "A ferramenta traduz a expressão em linguagem simples para que você possa confirmar seu significado sem memorizar a sintaxe",
    how3_t: "Visualize as próximas execuções",
    how3: "Alterne para a aba Resultados para visualizar as próximas datas de execução agendadas a partir de qualquer data de referência",
    how4_t: "Carregue mais datas",
    how4: "Clique em 'Adicionar mais resultados' para carregar mais datas futuras e verificar o comportamento de agendamento a longo prazo",
    t_real: "Exemplos Reais de Expressões Cron",
    real_intro: "Cenários comuns de agendamento e as expressões CRON exatas que os alimentam",
    real1: "0 2 * * * — Backup diário do banco de dados às 2h da manhã",
    real2: "*/15 * * * * — Verificação de saúde a cada 15 minutos",
    real3: "0 9 * * MON — E-mail de relatório semanal toda segunda-feira às 9h",
    real4: "0 0 1 1 * — Trabalho de arquivamento anual em 1º de janeiro à meia-noite",
    real5: "0 18 * * FRI — Resumo de fim de semana toda sexta-feira às 18h",
    t_edge: "Casos Especiais e Padrões Avançados",
    edge_intro: "Cenários de agendamento difíceis que a maioria das ferramentas não explica",
    edge1: "0,30 * * * * — Executa nos minutos 0 e 30 de cada hora, ou seja, duas vezes por hora",
    edge2: "30 0 2 * * * — Expressão de 6 campos: executa exatamente às 02:00:30 (precisão de segundos)",
    edge3: "0 8-18 * * 1-5 — Executa a cada hora das 8h às 18h, apenas de segunda a sexta-feira",
    t_faq: "Perguntas Frequentes",
    faq1q: "O que é uma expressão crontab?",
    faq1a: "Uma expressão crontab é uma string de 5 ou 6 campos separados por espaços que define quando uma tarefa agendada (cron job) deve ser executada em sistemas Unix. Cada campo representa uma unidade de tempo: minuto, hora, dia do mês, mês e dia da semana — com um campo de segundos opcional no início.",
    faq2q: "Qual é a diferença entre expressões cron de 5 e 6 campos?",
    faq2a: "Uma expressão de 5 campos começa no nível de minuto (minuto hora dia mês dia-da-semana), enquanto uma de 6 campos adiciona um campo de segundos no início (segundo minuto hora dia mês dia-da-semana). O cron de 6 campos é usado por agendadores como o Quartz (Java) e alguns executores de tarefas modernos.",
    faq3q: "Como executo uma tarefa a cada 5 minutos?",
    faq3a: "Use a expressão */5 * * * * que significa 'a cada 5 minutos'. A sintaxe */5 significa 'a cada passo de 5 a partir de 0', então a tarefa roda nos minutos 0, 5, 10, 15, e assim por diante.",
    faq4q: "O que significa */2 em uma expressão cron?",
    faq4a: "A notação */2 significa 'a cada 2 unidades'. No campo de minutos significa a cada 2 minutos (0, 2, 4, …). No campo de horas significa a cada 2 horas (0, 2, 4, …). A barra / define um valor de passo sobre o intervalo completo.",
    faq5q: "Posso usar nomes de meses e dias da semana em vez de números?",
    faq5a: "Sim. Você pode usar JAN a DEC para meses e SUN a SAT (ou MON-FRI) para dias da semana. Os nomes não diferenciam maiúsculas de minúsculas na maioria dos sistemas. Por exemplo, 0 9 * * MON-FRI executa às 9h em todos os dias de semana.",
    faq6q: "Existe uma maneira de visualizar quando meu cron job será executado?",
    faq6a: "Sim — alterne para a aba Resultados nesta ferramenta. Ela calcula e lista as próximas datas de execução com base na sua expressão e em uma data de início de referência. Você também pode alterar a data de referência para simular o agendamento a partir de qualquer ponto no tempo.",
    feat1: "Suporta expressões CRON de 5 e 6 campos",
    feat2: "Validação em tempo real e descrição em linguagem natural",
    feat3: "Visualização das próximas datas agendadas a partir de qualquer data de referência",
    feat4: "Precisão de agendamento no nível de segundos",
    feat5: "Suporte a caracteres especiais L, # e intervalos",
    feat6: "Disponível em 6 idiomas"
  },
  es: {
    title: "Generador de Crontab",
    pageTitle: "Generador de Crontab — Expresiones CRON de 5 y 6 Campos",
    meta: "Generador de crontab online y gratuito. Crea y valida expresiones CRON con 5 o 6 campos al instante — visualiza las próximas fechas programadas desde cualquier fecha de referencia.",
    see1: "Generador de UUID",
    see2: "Letras Especiales",
    see3: "Generador de Tarjeta de Crédito",
    see4: "Generador de Fecha de Nacimiento"
  },
  fr: {
    title: "Générateur de Crontab",
    pageTitle: "Générateur de Crontab — Expressions CRON à 5 et 6 Champs",
    meta: "Générateur de crontab en ligne et gratuit. Créez et validez des expressions CRON à 5 ou 6 champs instantanément — visualisez les prochaines dates planifiées depuis n'importe quelle date de référence.",
    see1: "Générateur UUID",
    see2: "Lettres Fantaisie",
    see3: "Générateur de Carte de Crédit",
    see4: "Générateur de Date de Naissance"
  },
  it: {
    title: "Generatore di Crontab",
    pageTitle: "Generatore di Crontab — Espressioni CRON a 5 e 6 Campi",
    meta: "Generatore di crontab online e gratuito. Crea e valida espressioni CRON con 5 o 6 campi istantaneamente — visualizza le prossime date pianificate da qualsiasi data di riferimento.",
    see1: "Generatore UUID",
    see2: "Lettere Particolari",
    see3: "Generatore di Carte di Credito",
    see4: "Generatore di Data di Nascita"
  },
  id: {
    title: "Generator Crontab",
    pageTitle: "Generator Crontab — Ekspresi CRON 5 dan 6 Bidang",
    meta: "Generator crontab online dan gratis. Buat dan validasi ekspresi CRON dengan 5 atau 6 bidang secara instan — lihat tanggal terjadwal berikutnya dari tanggal referensi mana pun.",
    see1: "Generator UUID",
    see2: "Huruf Mewah",
    see3: "Generator Kartu Kredit",
    see4: "Generator Tanggal Lahir"
  }
}
</i18n>
