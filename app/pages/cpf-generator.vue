<script setup lang="ts">
const { t } = useI18n({ useScope: 'local' })

const regions = Array.from({ length: 10 }, (_, i) => i.toString())

interface CpfItem {
  cpf: string
  region: string
}

const state = reactive({
  cpf: '',
  justNum: false,
  cpfList: [] as CpfItem[],
  multipleCount: 5,
  showMultiple: false,
  selectedRegion: 'all',
  generatedRegion: '',
  ads: false
})

const randomDigit = (): number => Math.floor(Math.random() * 10)

function getRegionDigit(): number {
  if (state.selectedRegion === 'all') return randomDigit()
  return parseInt(state.selectedRegion)
}

function formatCPF(digits: number[], justNum: boolean): string {
  const joined = digits.join('')
  return justNum
    ? joined
    : `${joined.slice(0, 3)}.${joined.slice(3, 6)}.${joined.slice(6, 9)}-${joined.slice(9)}`
}

function generateCPF(): { cpf: string; region: string } {
  const digits = Array.from({ length: 8 }, () => randomDigit())
  const regionDigit = getRegionDigit()
  digits.push(regionDigit)
  let sum = digits.reduce((acc, digit, index) => acc + digit * (10 - index), 0)
  const d1 = sum % 11 < 2 ? 0 : 11 - (sum % 11)
  sum = digits.reduce((acc, digit, index) => acc + digit * (11 - index), 0) + d1 * 2
  const d2 = sum % 11 < 2 ? 0 : 11 - (sum % 11)
  return {
    cpf: formatCPF([...digits, d1, d2], state.justNum),
    region: regionDigit.toString()
  }
}

function generate(): void {
  state.cpfList = []
  const result = generateCPF()
  state.cpf = result.cpf
  state.generatedRegion = result.region
  if (!state.ads) state.ads = true
}

function generateMultiple(): void {
  const count = Math.min(Math.max(state.multipleCount, 2), 100)
  state.cpf = ''
  state.generatedRegion = ''
  const cpfs = new Map<string, string>()
  let attempts = 0
  while (cpfs.size < count && attempts < count * 10) {
    const result = generateCPF()
    cpfs.set(result.cpf, result.region)
    attempts++
  }
  state.cpfList = Array.from(cpfs.entries()).map(([cpf, region]) => ({ cpf, region }))
  if (!state.ads) state.ads = true
}

function copyAll(): void {
  const allCpfs = state.cpfList.map(item => item.cpf).join('\n')
  navigator.clipboard.writeText(allCpfs).catch(() => { })
}

function getRegionName(digit: string): string {
  return t(`region${digit}`)
}

watch(() => state.justNum, () => {
  if (state.cpf) {
    const numbers = state.cpf.replace(/\D/g, '')
    if (numbers.length === 11)
      state.cpf = formatCPF(numbers.split('').map(Number), state.justNum)
  }
  if (state.cpfList.length > 0) {
    state.cpfList = state.cpfList.map(item => {
      const numbers = item.cpf.replace(/\D/g, '')
      return { cpf: formatCPF(numbers.split('').map(Number), state.justNum), region: item.region }
    })
  }
})

watch(() => state.showMultiple, () => {
  if (state.showMultiple) {
    state.cpf = ''
    state.generatedRegion = ''
  } else {
    state.cpfList = []
  }
})

watch(() => state.selectedRegion, () => {
  state.cpf = ''
  state.generatedRegion = ''
  state.cpfList = []
})

// Gera o primeiro CPF assim que a página carrega no cliente.
// Reduz fricção para a intenção transacional ("quero um CPF agora")
// e melhora sinais de comportamento (interação imediata).
onMounted(() => {
  generate()
})

usePageJsonLd({
  name: t('title'),
  description: t('meta'),
  features: [t('f_1'), t('f_2'), t('f_3'), t('f_4')],
  faq: [
    { question: t('faq_1_q'), answer: t('faq_1_a') },
    { question: t('faq_2_q'), answer: t('faq_2_a') },
    { question: t('faq_3_q'), answer: t('faq_3_a') },
    { question: t('faq_4_q'), answer: t('faq_4_a') },
    { question: t('faq_5_q'), answer: t('faq_5_a') }
  ],
  howToName: t('how_to_use_title'),
  howToSteps: [
    { name: t('step_1_title'), text: t('step_1_desc') },
    { name: t('step_2_title'), text: t('step_2_desc') },
    { name: t('step_3_title'), text: t('step_3_desc') }
  ]
})

useHead({
  title: t('pageTitle'),
  meta: [{ name: 'description', content: t('meta') }]
})

defineI18nRoute({
  paths: {
    en: '/cpf-generator',
    pt: '/gerador-de-cpf',
    es: '/generador-de-cpf',
    fr: '/generateur-de-cpf',
    it: '/generatore-di-cpf',
    id: '/generator-cpf'
  }
})
</script>

<template>
  <ToolPage :title="t('title')" :description="t('meta')" :show-ads="state.ads"
    wiki-url="https://pt.wikipedia.org/wiki/Cadastro_de_pessoas_f%C3%ADsicas"
    wiki-label="CPF (Cadastro de Pessoas Físicas)" :see-also-links="[
      { label: t('see1') + ' 🇧🇷', to: 'cnpj-generator' },
      { label: t('see2'), to: 'fancy-letters' },
      { label: t('see3'), to: 'credit-card-generator' },
      { label: t('see4'), to: 'birthday-generator' }
    ]">
    <div class="grid lg:grid-cols-2 gap-8 mb-4">
      <!-- Controles -->
      <div class="space-y-5">
        <div class="flex flex-col gap-3">
          <label class="flex items-center gap-3 cursor-pointer">
            <input type="checkbox" v-model="state.justNum" class="toggle toggle-primary" />
            <span class="font-medium">{{ t('num') }}</span>
          </label>
          <label class="flex items-center gap-3 cursor-pointer">
            <input type="checkbox" v-model="state.showMultiple" class="toggle toggle-primary" />
            <span class="font-medium">{{ t('multiple_label') }}</span>
          </label>
        </div>

        <div class="form-control">
          <label class="label pb-1" for="region-select">
            <span class="label-text font-bold text-base-content/80">{{ t('select_state') }}</span>
          </label>
          <select id="region-select" v-model="state.selectedRegion" class="select select-bordered w-full">
            <option value="all">{{ t('all_states') }}</option>
            <option v-for="r in regions" :key="r" :value="r">{{ t(`region${r}`) }}</option>
          </select>
        </div>

        <div v-if="state.showMultiple" class="form-control">
          <label class="label pb-1" for="qty-input">
            <span class="label-text font-bold text-base-content/80">{{ t('quantity') }}</span>
          </label>
          <input id="qty-input" type="number" v-model.number="state.multipleCount" min="2" max="100"
            class="input input-bordered w-full" />
        </div>

        <ButtonPrimary @click="state.showMultiple ? generateMultiple() : generate()"
          icon="heroicons:identification-20-solid" class="w-full h-14 text-lg">
          {{ state.showMultiple ? t('bt_multiple') : t('bt') }}
        </ButtonPrimary>

        <p class="text-sm text-base-content/50 italic px-1 text-center">{{ t('warning') }}</p>
      </div>

      <!-- Resultado -->
      <div class="bg-base-200/50 border border-primary/10 rounded-2xl p-6 min-h-[16rem] flex flex-col justify-center">
        <Transition enter-active-class="transition duration-300 ease-out"
          enter-from-class="transform scale-95 opacity-0" enter-to-class="transform scale-100 opacity-100">
          <div v-if="state.cpf && !state.cpfList.length" class="space-y-2 w-full">
            <LineCopy :content="state.cpf" :label="t('cpf_label')" class="!my-0 [&>div:last-child]:!w-full" />
            <p v-if="state.generatedRegion && state.selectedRegion === 'all'"
              class="text-xs text-base-content/60 px-1 mt-3">
              {{ t('region_from') }}: {{ getRegionName(state.generatedRegion) }}
            </p>
          </div>
        </Transition>

        <div v-if="state.cpfList.length > 0" class="w-full space-y-1">
          <div class="flex items-center justify-between mb-3">
            <span class="text-sm font-semibold">{{ t('generated_cpfs', { count: state.cpfList.length }) }}</span>
            <button @click="copyAll" class="btn btn-ghost btn-sm gap-1.5">
              <Icon name="material-symbols:content-copy-outline" class="w-4 h-4" />
              {{ t('copy_all') }}
            </button>
          </div>
          <div class="space-y-2 max-h-64 overflow-y-auto pr-1">
            <div v-for="(item, index) in state.cpfList" :key="index">
              <LineCopy :content="item.cpf" :label="`CPF ${index + 1}`" class="!my-0" />
              <p v-if="state.selectedRegion === 'all'" class="text-xs text-base-content/60 px-1 mt-2">{{
                getRegionName(item.region) }}</p>
            </div>
          </div>
        </div>

        <div v-if="!state.cpf && !state.cpfList.length" class="text-center opacity-70">
          <Icon name="heroicons:identification" class="w-16 h-16 mx-auto mb-2" />
          <p class="font-medium italic">{{ t('placeholder') }}</p>
        </div>
      </div>
    </div>

    <Mordizi v-if="state.ads" />

    <template #info>
      <div class="space-y-8">
        <section>
          <p>{{ t('d1') }}</p>
        </section>

        <FeatureSection :title="t('features_title')" :items="[t('f_1'), t('f_2'), t('f_3'), t('f_4')]" />

        <section>
          <h2 class="text-2xl font-bold mb-3 flex items-center gap-2">
            <Icon name="heroicons:book-open-20-solid" class="w-6 h-6 text-primary" />
            {{ t('how_title') }}
          </h2>
          <p class="text-base-content/80 leading-relaxed">{{ t('how_desc') }}</p>
        </section>

        <UseCaseSection :title="t('use_cases_title')" :description="t('use_cases_intro')" :items="[
          { title: t('use_1_t'), description: t('use_1_d') },
          { title: t('use_2_t'), description: t('use_2_d') },
          { title: t('use_3_t'), description: t('use_3_d') },
          { title: t('use_4_t'), description: t('use_4_d') },
          { title: t('use_5_t'), description: t('use_5_d') }
        ]" />

        <HowToSection :title="t('how_to_use_title')" :items="[
          { title: t('step_1_title'), description: t('step_1_desc') },
          { title: t('step_2_title'), description: t('step_2_desc') },
          { title: t('step_3_title'), description: t('step_3_desc') }
        ]" />

        <FaqSection :title="t('faq_title')" :items="[
          { question: t('faq_1_q'), answer: t('faq_1_a') },
          { question: t('faq_2_q'), answer: t('faq_2_a') },
          { question: t('faq_3_q'), answer: t('faq_3_a') },
          { question: t('faq_4_q'), answer: t('faq_4_a') },
          { question: t('faq_5_q'), answer: t('faq_5_a') }
        ]" />
      </div>
    </template>
  </ToolPage>
</template>

<i18n lang="yaml">
{
  pt: {
    pageTitle: "Gerador de CPF Válido Online — Grátis e Sem Cadastro",
    title: "Gerador de CPF Válido",
    meta: "Gere CPFs válidos online de graça. Por estado, em massa (até 100 por vez), com ou sem pontuação. Para testes de software e desenvolvimento.",

    num: "Somente Números",
    multiple_label: "Gerar Múltiplos CPFs",
    select_state: "Selecione o Estado",
    all_states: "Todos os Estados",
    region_from: "Região Fiscal",
    bt: "Gerar CPF",
    bt_multiple: "Gerar CPFs",
    cpf_label: "CPF Gerado",
    quantity: "Quantidade",
    generated_cpfs: "CPFs Gerados ({count})",
    copy_all: "Copiar Todos",
    placeholder: "Clique em Gerar CPF para criar um número",
    warning: "Os números gerados são fictícios e servem apenas para testes.",

    d1: "Esta ferramenta gera números de CPF aleatórios e matematicamente válidos, com os dígitos verificadores calculados corretamente. Os números servem para testar formulários, popular bases de homologação e validar regras de negócio em sistemas — qualquer cenário onde você precisa de um CPF que passe na validação do algoritmo, mas não está vinculado a uma pessoa real. Funciona direto no navegador, sem cadastro, e dá para filtrar por estado de origem ou gerar lotes de até 100 de uma vez.",

    features_title: "Funcionalidades",
    f_1: "Gera CPF válido por estado",
    f_2: "Lotes de até 100 CPFs únicos",
    f_3: "Formato com pontuação ou só números",
    f_4: "Dígitos verificadores calculados pelo módulo 11",

    how_title: "Como o algoritmo do CPF funciona",
    how_desc: "O CPF tem 11 dígitos: 9 de identificação e 2 verificadores no final. Os 9 primeiros são sequenciais por região fiscal, e o 9º indica o estado onde o documento foi emitido. Os dois últimos saem de um cálculo de módulo 11 sobre os anteriores: cada dígito é multiplicado por um peso, soma-se tudo, divide por 11, e o resto define o verificador. O método foi pensado para detectar erros de digitação. Trocar um dígito ou inverter dois quase sempre quebra a validação, o que permite a qualquer sistema rejeitar números inválidos sem precisar consultar a Receita.",

    use_cases_title: "Casos de Uso",
    use_cases_intro: "Onde um CPF gerado faz sentido no dia a dia de quem desenvolve:",
    use_1_t: "Validação de formulários",
    use_1_d: "Testar máscaras, validadores e mensagens de erro em campos de cadastro.",
    use_2_t: "Testes automatizados",
    use_2_d: "Popular cenários de teste com dados que passam na validação sem expor CPFs reais.",
    use_3_t: "Ambientes de desenvolvimento",
    use_3_d: "Criar registros em homologação e staging sem violar a LGPD nem usar dados de produção.",
    use_4_t: "Integrações e APIs",
    use_4_d: "Testar endpoints e serviços de terceiros que exigem CPF como parâmetro.",
    use_5_t: "Mockups e demonstrações",
    use_5_d: "Preencher protótipos, telas de demo e materiais de apresentação sem dados reais.",

    how_to_use_title: "Como Usar",
    step_1_title: "Gerar um CPF rápido",
    step_1_desc: "Ao abrir a página, o primeiro CPF já aparece pronto. Para gerar outro, clique em \"Gerar CPF\" — o número fica disponível para copiar com um clique.",
    step_2_title: "Gerar vários de uma vez",
    step_2_desc: "Marque \"Gerar Múltiplos CPFs\", escolha quantos (entre 2 e 100) e clique em \"Gerar CPFs\". O botão \"Copiar Todos\" passa a lista inteira para a área de transferência.",
    step_3_title: "Escolher o formato",
    step_3_desc: "Por padrão, o CPF vem com pontos e traço. Se você precisa do número limpo (banco de dados, JSON, etc.), ative \"Somente Números\" antes de gerar.",

    region0: "Rio Grande do Sul (RS)",
    region1: "Distrito Federal (DF), Goiás (GO), Mato Grosso (MT), Mato Grosso do Sul (MS), Tocantins (TO)",
    region2: "Acre (AC), Amapá (AP), Amazonas (AM), Pará (PA), Rondônia (RO), Roraima (RR)",
    region3: "Ceará (CE), Maranhão (MA), Piauí (PI)",
    region4: "Alagoas (AL), Paraíba (PB), Pernambuco (PE), Rio Grande do Norte (RN)",
    region5: "Bahia (BA), Sergipe (SE)",
    region6: "Minas Gerais (MG)",
    region7: "Espírito Santo (ES), Rio de Janeiro (RJ)",
    region8: "São Paulo (SP)",
    region9: "Paraná (PR), Santa Catarina (SC)",

    faq_title: "Perguntas Frequentes",
    faq_1_q: "Os CPFs gerados aqui são reais ou estão registrados na Receita Federal?",
    faq_1_a: "Não. São números fictícios construídos com o mesmo algoritmo de validação que a Receita usa, mas sem vínculo nenhum com pessoas reais. Eles passam na verificação dos dígitos porque o cálculo está correto, e nada além disso. Não tente usar para cadastros oficiais — não vão ser aceitos, e a tentativa pode configurar crime.",
    faq_2_q: "Por que existe um campo de estado no gerador?",
    faq_2_a: "O 9º dígito do CPF (logo antes dos verificadores) corresponde à região fiscal onde o documento foi emitido. São Paulo é 8, Rio Grande do Sul é 0, e por aí vai. Para alguns testes, faz diferença gerar números coerentes com uma região específica, e o filtro existe para isso. Se não importa, deixe em \"Todos os Estados\" que o sorteio é aleatório.",
    faq_3_q: "Quando usar o formato com pontos e quando usar só números?",
    faq_3_a: "Depende de onde o número vai entrar. Formulários e telas que mostram o CPF para o usuário normalmente usam o formato pontuado (999.999.999-99). Já bancos de dados, APIs e validações de backend tendem a armazenar o número limpo, sem caracteres especiais. O toggle \"Somente Números\" resolve sem precisar tratar a string depois.",
    faq_4_q: "Posso usar o gerador para qualquer finalidade?",
    faq_4_a: "Para testes de software, estudo, demonstrações e desenvolvimento, sim — é exatamente para isso que ele existe. O que você não pode fazer é usar um CPF gerado em cadastros oficiais, contratos, declarações ou qualquer situação que exija identificação real. Isso configura falsidade ideológica (Art. 299 do Código Penal) e, dependendo do caso, estelionato.",
    faq_5_q: "Qual a diferença entre \"CPF fake\" e \"CPF aleatório\"?",
    faq_5_a: "Na prática, é a mesma coisa: um número que segue a regra matemática do CPF mas não pertence a ninguém. Os termos \"fake\" e \"falso\" são populares na busca, embora alguns desenvolvedores prefiram \"fictício\" ou \"de teste\" por soarem menos sugestivos. O resultado e o uso são idênticos.",

    see1: "Gerador de CNPJ",
    see2: "Letras Diferentes",
    see3: "Gerador de Cartão de Crédito",
    see4: "Gerador de Data de Nascimento"
  },
  en: {
  pageTitle: "Brazilian CPF Generator — Free Online Tool, No Sign-up",
  title: "Brazilian CPF Generator",
  meta: "Generate valid Brazilian CPF numbers online for free. Filter by state, create up to 100 at once, with or without punctuation. For software testing and development.",

  num: "Numbers Only",
  multiple_label: "Generate Multiple CPFs",
  select_state: "Select the State",
  all_states: "All States",
  region_from: "Tax Region",
  bt: "Generate CPF",
  bt_multiple: "Generate CPFs",
  cpf_label: "Generated CPF",
  quantity: "Quantity",
  generated_cpfs: "Generated CPFs ({count})",
  copy_all: "Copy All",
  placeholder: "Click Generate CPF to create a number",
  warning: "The numbers are fictitious and meant for testing purposes only.",

  d1: "This tool generates random, mathematically valid Brazilian CPF numbers, with check digits calculated according to the official algorithm. The numbers are useful for testing forms, populating staging databases, and validating business rules in software — any scenario where you need a CPF that passes algorithmic validation but is not tied to a real person. It runs entirely in the browser, requires no registration, and lets you filter by state of origin or generate batches of up to 100 at a time.",

  features_title: "Features",
  f_1: "Generate a valid CPF by Brazilian state",
  f_2: "Batches of up to 100 unique CPFs",
  f_3: "Punctuated format or numbers only",
  f_4: "Check digits calculated using modulo 11",

  how_title: "How the CPF Algorithm Works",
  how_desc: "A CPF has 11 digits: 9 for identification and 2 check digits at the end. The first 9 are sequential within each tax region, and the 9th specifically encodes the state where the document was issued. The last two come from a modulo 11 calculation applied over the previous digits: each digit is multiplied by a weight, the results are summed, divided by 11, and the remainder determines the check digit. The method was designed to catch typing errors. Changing one digit or swapping two of them almost always breaks validation, which lets any system reject invalid numbers without querying the tax authority.",

  use_cases_title: "Use Cases",
  use_cases_intro: "Where a generated CPF makes sense in a developer's day-to-day work:",
  use_1_t: "Form Validation",
  use_1_d: "Test masks, validators, and error messages on registration fields.",
  use_2_t: "Automated Testing",
  use_2_d: "Populate test scenarios with data that passes validation without exposing real CPFs.",
  use_3_t: "Development Environments",
  use_3_d: "Create records in staging and homologation without violating privacy laws or copying production data.",
  use_4_t: "API Integration",
  use_4_d: "Test endpoints and third-party services that require a CPF as input.",
  use_5_t: "Mockups and Demos",
  use_5_d: "Fill prototypes, demo screens, and presentation materials with believable but non-real data.",

  how_to_use_title: "How to Use",
  step_1_title: "Generate a single CPF",
  step_1_desc: "When the page loads, the first CPF is already generated. To get another one, click \"Generate CPF\" — the number can be copied with a single click.",
  step_2_title: "Generate several at once",
  step_2_desc: "Enable \"Generate Multiple CPFs\", choose how many (between 2 and 100), and click \"Generate CPFs\". The \"Copy All\" button sends the full list to the clipboard.",
  step_3_title: "Choose the format",
  step_3_desc: "By default, the CPF is returned with dots and a dash. If you need the raw number (databases, JSON payloads, etc.), enable \"Numbers Only\" before generating.",

  region0: "Rio Grande do Sul (RS)",
  region1: "Federal District (DF), Goiás (GO), Mato Grosso (MT), Mato Grosso do Sul (MS), Tocantins (TO)",
  region2: "Acre (AC), Amapá (AP), Amazonas (AM), Pará (PA), Rondônia (RO), Roraima (RR)",
  region3: "Ceará (CE), Maranhão (MA), Piauí (PI)",
  region4: "Alagoas (AL), Paraíba (PB), Pernambuco (PE), Rio Grande do Norte (RN)",
  region5: "Bahia (BA), Sergipe (SE)",
  region6: "Minas Gerais (MG)",
  region7: "Espírito Santo (ES), Rio de Janeiro (RJ)",
  region8: "São Paulo (SP)",
  region9: "Paraná (PR), Santa Catarina (SC)",

  faq_title: "Frequently Asked Questions",
  faq_1_q: "Are the generated CPFs real or registered with Brazil's tax authority?",
  faq_1_a: "No. They are fictitious numbers built with the same validation algorithm the Receita Federal uses, but with no link to real individuals. They pass the check-digit verification because the math is correct, and nothing beyond that. Do not attempt to use them in official records — they will not be accepted, and the attempt may constitute a criminal offense under Brazilian law.",
  faq_2_q: "Why is there a state field in the generator?",
  faq_2_a: "The 9th digit of a CPF (right before the check digits) corresponds to the tax region where the document was originally issued. São Paulo is 8, Rio Grande do Sul is 0, and so on. For some testing scenarios, generating numbers consistent with a specific region matters, which is what the filter is for. If it does not, leave it on \"All States\" and the region will be picked at random.",
  faq_3_q: "When should I use the punctuated format versus numbers only?",
  faq_3_a: "It depends on where the number is going. User-facing forms and screens normally display the punctuated format (999.999.999-99). Databases, APIs, and backend validators typically store the raw number with no special characters. The \"Numbers Only\" toggle handles this directly, so you don't need to strip the string afterwards.",
  faq_4_q: "Is it legal to use this generator?",
  faq_4_a: "For software testing, study, demonstrations, and development work, yes — that is precisely what it is built for. What you cannot do is use a generated CPF in official registrations, contracts, declarations, or any situation that requires real identification. Doing so qualifies as ideological falsehood under Article 299 of the Brazilian Penal Code, and may also constitute fraud depending on the context.",
  faq_5_q: "What is the difference between \"CPF generator\", \"fake CPF\", and \"random CPF\"?",
  faq_5_a: "In practice, they refer to the same thing: a number that follows the mathematical rules of a CPF but does not belong to anyone. The terms \"fake\" and \"fictitious\" are interchangeable in search queries, though developers tend to prefer \"test CPF\" or \"fictitious CPF\" in technical contexts. The output and the appropriate use cases are identical.",

  see1: "CNPJ Generator",
  see2: "Fancy Letters",
  see3: "Credit Card Generator",
  see4: "Birthday Generator"
},
es: {
  pageTitle: "Generador de CPF Válido Online — Gratis y Sin Registro",
  title: "Generador de CPF Válido",
  meta: "Genera números de CPF brasileños válidos online y gratis. Filtra por estado, crea hasta 100 a la vez, con o sin puntuación. Para pruebas de software y desarrollo.",

  num: "Solo Números",
  multiple_label: "Generar Múltiples CPFs",
  select_state: "Selecciona el Estado",
  all_states: "Todos los Estados",
  region_from: "Región Fiscal",
  bt: "Generar CPF",
  bt_multiple: "Generar CPFs",
  cpf_label: "CPF Generado",
  quantity: "Cantidad",
  generated_cpfs: "CPFs Generados ({count})",
  copy_all: "Copiar Todos",
  placeholder: "Haz clic en Generar CPF para crear un número",
  warning: "Los números generados son ficticios y se destinan únicamente a pruebas.",

  d1: "Esta herramienta genera números de CPF brasileños aleatorios y matemáticamente válidos, con los dígitos verificadores calculados según el algoritmo oficial. Los números sirven para probar formularios, poblar bases de homologación y validar reglas de negocio en sistemas — cualquier escenario en el que se necesite un CPF que supere la validación algorítmica pero no esté vinculado a una persona real. Funciona directamente en el navegador, sin registro, y permite filtrar por estado de origen o generar lotes de hasta 100 a la vez.",

  features_title: "Funcionalidades",
  f_1: "Genera un CPF válido por estado brasileño",
  f_2: "Lotes de hasta 100 CPFs únicos",
  f_3: "Formato con puntuación o solo números",
  f_4: "Dígitos verificadores calculados con módulo 11",

  how_title: "Cómo Funciona el Algoritmo del CPF",
  how_desc: "El CPF tiene 11 dígitos: 9 de identificación y 2 verificadores al final. Los 9 primeros son secuenciales dentro de cada región fiscal, y el 9º indica específicamente el estado donde se emitió el documento. Los dos últimos resultan de un cálculo de módulo 11 sobre los dígitos anteriores: cada uno se multiplica por un peso, se suman los resultados, se divide entre 11, y el resto determina el dígito verificador. El método fue diseñado para detectar errores de digitación. Cambiar un dígito o invertir dos de ellos casi siempre rompe la validación, lo que permite a cualquier sistema rechazar números inválidos sin consultar a la administración tributaria.",

  use_cases_title: "Casos de Uso",
  use_cases_intro: "Dónde tiene sentido un CPF generado en el día a día del desarrollo:",
  use_1_t: "Validación de Formularios",
  use_1_d: "Probar máscaras, validadores y mensajes de error en campos de registro.",
  use_2_t: "Pruebas Automatizadas",
  use_2_d: "Poblar escenarios de prueba con datos que pasan la validación sin exponer CPFs reales.",
  use_3_t: "Entornos de Desarrollo",
  use_3_d: "Crear registros en homologación y staging sin violar leyes de protección de datos ni usar información de producción.",
  use_4_t: "Integración de APIs",
  use_4_d: "Probar endpoints y servicios de terceros que requieren un CPF como parámetro.",
  use_5_t: "Mockups y Demostraciones",
  use_5_d: "Rellenar prototipos, pantallas de demo y materiales de presentación con datos verosímiles pero no reales.",

  how_to_use_title: "Cómo Usar",
  step_1_title: "Generar un CPF rápido",
  step_1_desc: "Al abrir la página, el primer CPF ya aparece listo. Para generar otro, haz clic en \"Generar CPF\" — el número puede copiarse con un solo clic.",
  step_2_title: "Generar varios a la vez",
  step_2_desc: "Activa \"Generar Múltiples CPFs\", elige cuántos (entre 2 y 100) y haz clic en \"Generar CPFs\". El botón \"Copiar Todos\" envía la lista completa al portapapeles.",
  step_3_title: "Elegir el formato",
  step_3_desc: "Por defecto, el CPF se entrega con puntos y guion. Si necesitas el número sin formato (bases de datos, JSON, etc.), activa \"Solo Números\" antes de generar.",

  region0: "Rio Grande do Sul (RS)",
  region1: "Distrito Federal (DF), Goiás (GO), Mato Grosso (MT), Mato Grosso do Sul (MS), Tocantins (TO)",
  region2: "Acre (AC), Amapá (AP), Amazonas (AM), Pará (PA), Rondônia (RO), Roraima (RR)",
  region3: "Ceará (CE), Maranhão (MA), Piauí (PI)",
  region4: "Alagoas (AL), Paraíba (PB), Pernambuco (PE), Rio Grande do Norte (RN)",
  region5: "Bahia (BA), Sergipe (SE)",
  region6: "Minas Gerais (MG)",
  region7: "Espírito Santo (ES), Rio de Janeiro (RJ)",
  region8: "São Paulo (SP)",
  region9: "Paraná (PR), Santa Catarina (SC)",

  faq_title: "Preguntas Frecuentes",
  faq_1_q: "¿Los CPFs generados son reales o están registrados en la Receita Federal de Brasil?",
  faq_1_a: "No. Son números ficticios construidos con el mismo algoritmo de validación que utiliza la Receita Federal, pero sin vínculo alguno con personas reales. Pasan la verificación de los dígitos verificadores porque el cálculo es correcto, y nada más allá de eso. No intentes usarlos en registros oficiales — no serán aceptados, y el intento puede configurar un delito según la legislación brasileña.",
  faq_2_q: "¿Por qué hay un campo de estado en el generador?",
  faq_2_a: "El 9º dígito del CPF (justo antes de los verificadores) corresponde a la región fiscal donde se emitió originalmente el documento. São Paulo es 8, Rio Grande do Sul es 0, y así sucesivamente. Para ciertos escenarios de prueba, importa generar números coherentes con una región específica, y el filtro existe para eso. Si no es relevante, déjalo en \"Todos los Estados\" y la región se elegirá al azar.",
  faq_3_q: "¿Cuándo usar el formato con puntuación y cuándo solo números?",
  faq_3_a: "Depende de dónde vaya a ingresarse el número. Los formularios y pantallas que muestran el CPF al usuario normalmente usan el formato con puntuación (999.999.999-99). En cambio, las bases de datos, las APIs y los validadores de backend suelen almacenar el número sin caracteres especiales. El interruptor \"Solo Números\" resuelve esto sin necesidad de procesar la cadena después.",
  faq_4_q: "¿Es legal usar este generador?",
  faq_4_a: "Para pruebas de software, estudio, demostraciones y desarrollo, sí — es exactamente para eso que existe. Lo que no se puede hacer es usar un CPF generado en registros oficiales, contratos, declaraciones o cualquier situación que requiera identificación real. Eso configura falsedad ideológica (Art. 299 del Código Penal Brasileño) y, según el caso, también estafa.",
  faq_5_q: "¿Cuál es la diferencia entre \"generador de CPF\", \"CPF falso\" y \"CPF aleatorio\"?",
  faq_5_a: "En la práctica, se refieren a lo mismo: un número que sigue las reglas matemáticas de un CPF pero no pertenece a nadie. Los términos \"falso\" y \"ficticio\" son intercambiables en las búsquedas, aunque los desarrolladores suelen preferir \"CPF de prueba\" o \"CPF ficticio\" en contextos técnicos. El resultado y los usos apropiados son idénticos.",

  see1: "Generador de CNPJ",
  see2: "Letras Diferentes",
  see3: "Generador de Tarjeta de Crédito",
  see4: "Generador de Fecha de Nacimiento"
  },
  fr: {
  pageTitle: "Générateur de CPF Valide en Ligne — Gratuit et Sans Inscription",
  title: "Générateur de CPF Valide",
  meta: "Générez des numéros de CPF brésiliens valides en ligne et gratuitement. Filtrez par État, créez jusqu'à 100 à la fois, avec ou sans ponctuation. Pour les tests logiciels et le développement.",

  num: "Chiffres Uniquement",
  multiple_label: "Générer Plusieurs CPF",
  select_state: "Sélectionnez l'État",
  all_states: "Tous les États",
  region_from: "Région Fiscale",
  bt: "Générer un CPF",
  bt_multiple: "Générer des CPF",
  cpf_label: "CPF Généré",
  quantity: "Quantité",
  generated_cpfs: "CPF Générés ({count})",
  copy_all: "Tout Copier",
  placeholder: "Cliquez sur Générer un CPF pour créer un numéro",
  warning: "Les numéros générés sont fictifs et destinés exclusivement aux tests.",

  d1: "Cet outil génère des numéros de CPF brésiliens aléatoires et mathématiquement valides, avec les chiffres de contrôle calculés selon l'algorithme officiel. Les numéros servent à tester des formulaires, alimenter des bases de pré-production et valider des règles métier dans des systèmes — tout scénario nécessitant un CPF qui passe la validation algorithmique sans être lié à une personne réelle. L'outil fonctionne directement dans le navigateur, sans inscription, et permet de filtrer par État d'origine ou de générer des lots allant jusqu'à 100 numéros à la fois.",

  features_title: "Fonctionnalités",
  f_1: "Génère un CPF valide par État brésilien",
  f_2: "Lots allant jusqu'à 100 CPF uniques",
  f_3: "Format avec ponctuation ou chiffres uniquement",
  f_4: "Chiffres de contrôle calculés via le modulo 11",

  how_title: "Fonctionnement de l'Algorithme du CPF",
  how_desc: "Un CPF comporte 11 chiffres : 9 d'identification et 2 chiffres de contrôle à la fin. Les 9 premiers sont séquentiels au sein de chaque région fiscale, et le 9e indique précisément l'État dans lequel le document a été délivré. Les deux derniers résultent d'un calcul de modulo 11 appliqué aux chiffres précédents : chaque chiffre est multiplié par un poids, les résultats sont additionnés, divisés par 11, et le reste détermine le chiffre de contrôle. La méthode a été conçue pour détecter les erreurs de saisie. Modifier un chiffre ou en intervertir deux suffit presque toujours à invalider le numéro, ce qui permet à n'importe quel système de rejeter les valeurs incorrectes sans interroger l'administration fiscale.",

  use_cases_title: "Cas d'Utilisation",
  use_cases_intro: "Où un CPF généré trouve sa place dans le quotidien d'un développeur :",
  use_1_t: "Validation de Formulaires",
  use_1_d: "Tester les masques de saisie, les validateurs et les messages d'erreur sur les champs d'inscription.",
  use_2_t: "Tests Automatisés",
  use_2_d: "Alimenter des scénarios de test avec des données qui passent la validation sans exposer de CPF réels.",
  use_3_t: "Environnements de Développement",
  use_3_d: "Créer des enregistrements en pré-production sans enfreindre les lois sur la protection des données ni copier des informations de production.",
  use_4_t: "Intégration d'API",
  use_4_d: "Tester les endpoints et les services tiers qui requièrent un CPF en paramètre.",
  use_5_t: "Maquettes et Démonstrations",
  use_5_d: "Remplir les prototypes, écrans de démo et supports de présentation avec des données crédibles mais non réelles.",

  how_to_use_title: "Mode d'Emploi",
  step_1_title: "Générer un CPF rapidement",
  step_1_desc: "Au chargement de la page, un premier CPF apparaît déjà. Pour en générer un autre, cliquez sur « Générer un CPF » — le numéro peut être copié d'un seul clic.",
  step_2_title: "Générer plusieurs numéros à la fois",
  step_2_desc: "Activez « Générer Plusieurs CPF », choisissez le nombre souhaité (entre 2 et 100), puis cliquez sur « Générer des CPF ». Le bouton « Tout Copier » envoie la liste complète dans le presse-papiers.",
  step_3_title: "Choisir le format",
  step_3_desc: "Par défaut, le CPF est affiché avec points et tiret. Si vous avez besoin du numéro brut (bases de données, JSON, etc.), activez « Chiffres Uniquement » avant de générer.",

  region0: "Rio Grande do Sul (RS)",
  region1: "District Fédéral (DF), Goiás (GO), Mato Grosso (MT), Mato Grosso do Sul (MS), Tocantins (TO)",
  region2: "Acre (AC), Amapá (AP), Amazonas (AM), Pará (PA), Rondônia (RO), Roraima (RR)",
  region3: "Ceará (CE), Maranhão (MA), Piauí (PI)",
  region4: "Alagoas (AL), Paraíba (PB), Pernambuco (PE), Rio Grande do Norte (RN)",
  region5: "Bahia (BA), Sergipe (SE)",
  region6: "Minas Gerais (MG)",
  region7: "Espírito Santo (ES), Rio de Janeiro (RJ)",
  region8: "São Paulo (SP)",
  region9: "Paraná (PR), Santa Catarina (SC)",

  faq_title: "Questions Fréquentes",
  faq_1_q: "Les CPF générés sont-ils réels ou enregistrés auprès de l'administration fiscale brésilienne ?",
  faq_1_a: "Non. Ce sont des numéros fictifs construits avec le même algorithme de validation que celui utilisé par la Receita Federal, mais sans aucun lien avec des personnes réelles. Ils passent la vérification des chiffres de contrôle parce que le calcul est correct, et rien de plus. N'essayez pas de les utiliser pour des démarches officielles — ils ne seront pas acceptés, et la tentative peut constituer une infraction pénale au regard du droit brésilien.",
  faq_2_q: "Pourquoi y a-t-il un champ « État » dans le générateur ?",
  faq_2_a: "Le 9e chiffre du CPF (juste avant les chiffres de contrôle) correspond à la région fiscale dans laquelle le document a été initialement délivré. São Paulo correspond à 8, Rio Grande do Sul à 0, et ainsi de suite. Pour certains scénarios de test, il est utile que les numéros soient cohérents avec une région précise, et c'est à cela que sert le filtre. Si cela n'a pas d'importance, laissez « Tous les États » et la région sera tirée au hasard.",
  faq_3_q: "Quand utiliser le format avec ponctuation et quand utiliser les chiffres uniquement ?",
  faq_3_a: "Cela dépend de la destination du numéro. Les formulaires et écrans visibles par l'utilisateur affichent généralement le format avec ponctuation (999.999.999-99). En revanche, les bases de données, les API et les validateurs côté serveur stockent habituellement le numéro brut, sans caractères spéciaux. L'option « Chiffres Uniquement » règle ce point sans avoir à traiter la chaîne par la suite.",
  faq_4_q: "Est-il légal d'utiliser ce générateur ?",
  faq_4_a: "Pour les tests logiciels, l'étude, les démonstrations et le développement, oui — c'est précisément l'usage pour lequel il a été conçu. En revanche, il est interdit d'utiliser un CPF généré dans des inscriptions officielles, des contrats, des déclarations ou toute situation exigeant une identification réelle. Cela relève du faux idéologique (Art. 299 du Code Pénal brésilien) et, selon le contexte, peut également constituer une escroquerie.",
  faq_5_q: "Quelle est la différence entre « générateur de CPF », « CPF fake » et « CPF aléatoire » ?",
  faq_5_a: "En pratique, ces termes désignent la même chose : un numéro qui suit les règles mathématiques d'un CPF mais qui n'appartient à personne. Les expressions « fake » et « fictif » sont interchangeables dans les recherches, bien que les développeurs préfèrent souvent « CPF de test » ou « CPF fictif » dans un contexte technique. Le résultat et les usages appropriés restent identiques.",

  see1: "Générateur de CNPJ",
  see2: "Lettres Différentes",
  see3: "Générateur de Carte de Crédit",
  see4: "Générateur de Date de Naissance"
},
it: {
  pageTitle: "Generatore di CPF Valido Online — Gratis e Senza Registrazione",
  title: "Generatore di CPF Valido",
  meta: "Genera numeri di CPF brasiliani validi online e gratis. Filtra per Stato, crea fino a 100 numeri per volta, con o senza punteggiatura. Per test software e sviluppo.",

  num: "Solo Numeri",
  multiple_label: "Genera Più CPF",
  select_state: "Seleziona lo Stato",
  all_states: "Tutti gli Stati",
  region_from: "Regione Fiscale",
  bt: "Genera CPF",
  bt_multiple: "Genera CPF",
  cpf_label: "CPF Generato",
  quantity: "Quantità",
  generated_cpfs: "CPF Generati ({count})",
  copy_all: "Copia Tutti",
  placeholder: "Clicca su Genera CPF per creare un numero",
  warning: "I numeri generati sono fittizi e destinati esclusivamente ai test.",

  d1: "Questo strumento genera numeri di CPF brasiliani casuali e matematicamente validi, con i numeri di controllo calcolati secondo l'algoritmo ufficiale. I numeri servono per testare moduli, popolare database di staging e validare regole di business nei sistemi — qualsiasi scenario in cui serva un CPF che superi la validazione algoritmica senza essere collegato a una persona reale. Funziona direttamente nel browser, senza registrazione, e permette di filtrare per Stato di origine o di generare lotti fino a 100 numeri alla volta.",

  features_title: "Funzionalità",
  f_1: "Genera un CPF valido per Stato brasiliano",
  f_2: "Lotti fino a 100 CPF unici",
  f_3: "Formato con punteggiatura o solo numeri",
  f_4: "Numeri di controllo calcolati con il modulo 11",

  how_title: "Come Funziona l'Algoritmo del CPF",
  how_desc: "Un CPF è composto da 11 cifre: 9 di identificazione e 2 di controllo alla fine. Le prime 9 sono sequenziali all'interno di ciascuna regione fiscale, e la nona indica specificamente lo Stato in cui il documento è stato emesso. Le ultime due derivano da un calcolo di modulo 11 applicato sulle cifre precedenti: ogni cifra viene moltiplicata per un peso, i risultati vengono sommati, divisi per 11, e il resto determina la cifra di controllo. Il metodo è stato progettato per individuare errori di digitazione. Modificare una cifra o invertirne due basta quasi sempre a invalidare il numero, il che permette a qualsiasi sistema di rifiutare valori non corretti senza interrogare l'amministrazione fiscale.",

  use_cases_title: "Casi d'Uso",
  use_cases_intro: "Dove un CPF generato trova spazio nella quotidianità di chi sviluppa:",
  use_1_t: "Validazione di Moduli",
  use_1_d: "Testare maschere di input, validatori e messaggi di errore nei campi di registrazione.",
  use_2_t: "Test Automatizzati",
  use_2_d: "Popolare scenari di test con dati che superano la validazione senza esporre CPF reali.",
  use_3_t: "Ambienti di Sviluppo",
  use_3_d: "Creare record in staging e pre-produzione senza violare le leggi sulla protezione dei dati né copiare informazioni di produzione.",
  use_4_t: "Integrazione di API",
  use_4_d: "Testare endpoint e servizi di terze parti che richiedono un CPF come parametro.",
  use_5_t: "Mockup e Demo",
  use_5_d: "Riempire prototipi, schermate dimostrative e materiali di presentazione con dati credibili ma non reali.",

  how_to_use_title: "Come Usare",
  step_1_title: "Generare un CPF rapidamente",
  step_1_desc: "All'apertura della pagina, il primo CPF è già pronto. Per generarne un altro, clicca su «Genera CPF» — il numero può essere copiato con un solo clic.",
  step_2_title: "Generare più numeri in una volta",
  step_2_desc: "Attiva «Genera Più CPF», scegli la quantità (tra 2 e 100) e clicca su «Genera CPF». Il pulsante «Copia Tutti» invia l'elenco completo negli appunti.",
  step_3_title: "Scegliere il formato",
  step_3_desc: "Per impostazione predefinita, il CPF viene fornito con punti e trattino. Se serve il numero grezzo (database, JSON, ecc.), attiva «Solo Numeri» prima di generare.",

  region0: "Rio Grande do Sul (RS)",
  region1: "Distretto Federale (DF), Goiás (GO), Mato Grosso (MT), Mato Grosso do Sul (MS), Tocantins (TO)",
  region2: "Acre (AC), Amapá (AP), Amazonas (AM), Pará (PA), Rondônia (RO), Roraima (RR)",
  region3: "Ceará (CE), Maranhão (MA), Piauí (PI)",
  region4: "Alagoas (AL), Paraíba (PB), Pernambuco (PE), Rio Grande do Norte (RN)",
  region5: "Bahia (BA), Sergipe (SE)",
  region6: "Minas Gerais (MG)",
  region7: "Espírito Santo (ES), Rio de Janeiro (RJ)",
  region8: "São Paulo (SP)",
  region9: "Paraná (PR), Santa Catarina (SC)",

  faq_title: "Domande Frequenti",
  faq_1_q: "I CPF generati sono reali o registrati presso l'amministrazione fiscale brasiliana?",
  faq_1_a: "No. Sono numeri fittizi costruiti con lo stesso algoritmo di validazione utilizzato dalla Receita Federal, ma senza alcun collegamento con persone reali. Superano la verifica dei numeri di controllo perché il calcolo è corretto, e nient'altro. Non utilizzarli in registrazioni ufficiali — non verranno accettati, e il tentativo può configurare un reato secondo il diritto brasiliano.",
  faq_2_q: "Perché esiste un campo «Stato» nel generatore?",
  faq_2_a: "La nona cifra del CPF (subito prima dei numeri di controllo) corrisponde alla regione fiscale in cui il documento è stato inizialmente emesso. San Paolo corrisponde a 8, Rio Grande do Sul a 0, e così via. Per alcuni scenari di test è utile generare numeri coerenti con una regione specifica, ed è a questo che serve il filtro. Se non ha rilevanza, lascia «Tutti gli Stati» e la regione verrà scelta in modo casuale.",
  faq_3_q: "Quando usare il formato con punteggiatura e quando solo numeri?",
  faq_3_a: "Dipende da dove andrà inserito il numero. I moduli e le schermate visibili all'utente mostrano generalmente il formato con punteggiatura (999.999.999-99). Database, API e validatori lato server, invece, di solito memorizzano il numero grezzo, senza caratteri speciali. L'opzione «Solo Numeri» risolve la questione senza dover elaborare la stringa successivamente.",
  faq_4_q: "È legale utilizzare questo generatore?",
  faq_4_a: "Per test software, studio, dimostrazioni e attività di sviluppo, sì — è esattamente lo scopo per cui è stato creato. Ciò che non è consentito è utilizzare un CPF generato in registrazioni ufficiali, contratti, dichiarazioni o qualsiasi situazione che richieda un'identificazione reale. Configurare tale uso costituisce falso ideologico (Art. 299 del Codice Penale brasiliano) e, a seconda del contesto, può integrare anche il reato di truffa.",
  faq_5_q: "Qual è la differenza tra «generatore di CPF», «CPF fake» e «CPF casuale»?",
  faq_5_a: "In pratica, si riferiscono alla stessa cosa: un numero che segue le regole matematiche di un CPF ma non appartiene a nessuno. I termini «fake» e «fittizio» sono intercambiabili nelle ricerche, anche se gli sviluppatori tendono a preferire «CPF di test» o «CPF fittizio» in contesti tecnici. Il risultato e gli usi appropriati sono identici.",

  see1: "Generatore di CNPJ",
  see2: "Lettere Diverse",
  see3: "Generatore di Carte di Credito",
  see4: "Generatore di Data di Nascita"
},
id: {
  pageTitle: "Generator CPF Valid Online — Gratis Tanpa Pendaftaran",
  title: "Generator CPF Valid",
  meta: "Hasilkan nomor CPF Brasil yang valid secara online dan gratis. Filter berdasarkan negara bagian, buat hingga 100 sekaligus, dengan atau tanpa tanda baca. Untuk pengujian perangkat lunak dan pengembangan.",

  num: "Hanya Angka",
  multiple_label: "Hasilkan Beberapa CPF",
  select_state: "Pilih Negara Bagian",
  all_states: "Semua Negara Bagian",
  region_from: "Wilayah Pajak",
  bt: "Hasilkan CPF",
  bt_multiple: "Hasilkan CPF",
  cpf_label: "CPF Dihasilkan",
  quantity: "Jumlah",
  generated_cpfs: "CPF Dihasilkan ({count})",
  copy_all: "Salin Semua",
  placeholder: "Klik Hasilkan CPF untuk membuat nomor",
  warning: "Nomor yang dihasilkan bersifat fiktif dan hanya untuk keperluan pengujian.",

  d1: "Alat ini menghasilkan nomor CPF Brasil secara acak dan valid secara matematis, dengan digit pemeriksa yang dihitung sesuai algoritma resmi. Nomor-nomor tersebut berguna untuk menguji formulir, mengisi basis data staging, dan memvalidasi aturan bisnis pada sistem — skenario apa pun di mana Anda membutuhkan CPF yang lolos validasi algoritma tetapi tidak terkait dengan orang sungguhan. Alat ini berjalan langsung di peramban, tanpa pendaftaran, dan memungkinkan penyaringan berdasarkan negara bagian asal atau pembuatan hingga 100 nomor sekaligus.",

  features_title: "Fitur",
  f_1: "Menghasilkan CPF valid berdasarkan negara bagian Brasil",
  f_2: "Hingga 100 CPF unik dalam satu kali proses",
  f_3: "Format dengan tanda baca atau hanya angka",
  f_4: "Digit pemeriksa dihitung dengan modulo 11",

  how_title: "Cara Kerja Algoritma CPF",
  how_desc: "CPF terdiri dari 11 digit: 9 untuk identifikasi dan 2 sebagai digit pemeriksa di akhir. Sembilan digit pertama bersifat berurutan dalam setiap wilayah pajak, dan digit ke-9 secara khusus menunjukkan negara bagian tempat dokumen diterbitkan. Dua digit terakhir berasal dari perhitungan modulo 11 yang diterapkan pada digit-digit sebelumnya: setiap digit dikalikan dengan bobot tertentu, hasilnya dijumlahkan, dibagi 11, dan sisa pembagian menentukan digit pemeriksa. Metode ini dirancang untuk mendeteksi kesalahan pengetikan. Mengubah satu digit atau menukar dua digit hampir selalu menyebabkan validasi gagal, sehingga sistem mana pun dapat menolak nomor yang tidak valid tanpa perlu memeriksa ke otoritas pajak.",

  use_cases_title: "Kasus Penggunaan",
  use_cases_intro: "Situasi di mana CPF yang dihasilkan berguna dalam pekerjaan pengembangan sehari-hari:",
  use_1_t: "Validasi Formulir",
  use_1_d: "Menguji input mask, validator, dan pesan kesalahan pada kolom pendaftaran.",
  use_2_t: "Pengujian Otomatis",
  use_2_d: "Mengisi skenario pengujian dengan data yang lolos validasi tanpa mengekspos CPF asli.",
  use_3_t: "Lingkungan Pengembangan",
  use_3_d: "Membuat catatan di lingkungan staging dan homologasi tanpa melanggar undang-undang perlindungan data atau menyalin data produksi.",
  use_4_t: "Integrasi API",
  use_4_d: "Menguji endpoint dan layanan pihak ketiga yang membutuhkan CPF sebagai parameter.",
  use_5_t: "Mockup dan Demo",
  use_5_d: "Mengisi prototipe, layar demo, dan materi presentasi dengan data yang masuk akal namun bukan data asli.",

  how_to_use_title: "Cara Menggunakan",
  step_1_title: "Menghasilkan CPF dengan cepat",
  step_1_desc: "Saat halaman dimuat, CPF pertama sudah tersedia. Untuk menghasilkan nomor lain, klik \"Hasilkan CPF\" — nomor dapat disalin hanya dengan satu klik.",
  step_2_title: "Menghasilkan beberapa nomor sekaligus",
  step_2_desc: "Aktifkan \"Hasilkan Beberapa CPF\", pilih jumlah yang diinginkan (antara 2 dan 100), lalu klik \"Hasilkan CPF\". Tombol \"Salin Semua\" akan menyalin seluruh daftar ke papan klip.",
  step_3_title: "Memilih format",
  step_3_desc: "Secara bawaan, CPF ditampilkan dengan titik dan tanda hubung. Jika Anda membutuhkan nomor mentah (basis data, JSON, dan sebagainya), aktifkan \"Hanya Angka\" sebelum menghasilkan.",

  region0: "Rio Grande do Sul (RS)",
  region1: "Distrik Federal (DF), Goiás (GO), Mato Grosso (MT), Mato Grosso do Sul (MS), Tocantins (TO)",
  region2: "Acre (AC), Amapá (AP), Amazonas (AM), Pará (PA), Rondônia (RO), Roraima (RR)",
  region3: "Ceará (CE), Maranhão (MA), Piauí (PI)",
  region4: "Alagoas (AL), Paraíba (PB), Pernambuco (PE), Rio Grande do Norte (RN)",
  region5: "Bahia (BA), Sergipe (SE)",
  region6: "Minas Gerais (MG)",
  region7: "Espírito Santo (ES), Rio de Janeiro (RJ)",
  region8: "São Paulo (SP)",
  region9: "Paraná (PR), Santa Catarina (SC)",

  faq_title: "Pertanyaan yang Sering Diajukan",
  faq_1_q: "Apakah CPF yang dihasilkan ini asli atau terdaftar di otoritas pajak Brasil?",
  faq_1_a: "Tidak. Nomor-nomor tersebut bersifat fiktif dan dibuat menggunakan algoritma validasi yang sama dengan yang digunakan oleh Receita Federal, namun tidak memiliki kaitan apa pun dengan orang sungguhan. Nomor-nomor ini lolos verifikasi digit pemeriksa karena perhitungannya benar, hanya itu saja. Jangan mencoba menggunakannya untuk pendaftaran resmi — nomor tersebut tidak akan diterima, dan upaya tersebut dapat dianggap sebagai pelanggaran hukum di Brasil.",
  faq_2_q: "Mengapa terdapat kolom negara bagian pada generator?",
  faq_2_a: "Digit ke-9 pada CPF (tepat sebelum digit pemeriksa) menunjukkan wilayah pajak tempat dokumen pertama kali diterbitkan. São Paulo bernilai 8, Rio Grande do Sul bernilai 0, dan seterusnya. Untuk skenario pengujian tertentu, penting untuk menghasilkan nomor yang konsisten dengan wilayah tertentu, dan itulah fungsi filter ini. Jika tidak relevan, biarkan pada \"Semua Negara Bagian\" dan wilayah akan dipilih secara acak.",
  faq_3_q: "Kapan menggunakan format dengan tanda baca dan kapan hanya angka?",
  faq_3_a: "Tergantung pada tujuan penggunaan nomor tersebut. Formulir dan tampilan yang dilihat pengguna umumnya menggunakan format dengan tanda baca (999.999.999-99). Sementara itu, basis data, API, dan validator sisi server biasanya menyimpan nomor tanpa karakter khusus. Tombol \"Hanya Angka\" menyelesaikan hal ini secara langsung, tanpa perlu memproses string setelahnya.",
  faq_4_q: "Apakah legal menggunakan generator ini?",
  faq_4_a: "Untuk pengujian perangkat lunak, pembelajaran, demonstrasi, dan pekerjaan pengembangan, ya — itulah tujuan utamanya. Yang tidak diperbolehkan adalah menggunakan CPF yang dihasilkan untuk pendaftaran resmi, kontrak, deklarasi, atau situasi apa pun yang membutuhkan identifikasi asli. Tindakan tersebut termasuk pemalsuan ideologis (Pasal 299 Kitab Undang-Undang Hukum Pidana Brasil) dan, tergantung konteksnya, juga dapat dianggap sebagai penipuan.",
  faq_5_q: "Apa perbedaan antara \"generator CPF\", \"CPF palsu\", dan \"CPF acak\"?",
  faq_5_a: "Pada dasarnya, ketiganya mengacu pada hal yang sama: nomor yang mengikuti aturan matematis CPF tetapi tidak dimiliki oleh siapa pun. Istilah \"palsu\" dan \"fiktif\" dapat digunakan secara bergantian dalam pencarian, meskipun pengembang cenderung lebih memilih \"CPF uji\" atau \"CPF fiktif\" dalam konteks teknis. Hasil dan penggunaan yang sesuai pada akhirnya tetap sama.",

  see1: "Generator CNPJ",
  see2: "Huruf Berbeda",
  see3: "Generator Kartu Kredit",
  see4: "Generator Tanggal Lahir"
},
de: {
  pageTitle: "CPF-Generator Online — Kostenlos und ohne Anmeldung",
  title: "Gültiger CPF-Generator",
  meta: "Erzeugen Sie gültige brasilianische CPF-Nummern kostenlos online. Filtern nach Bundesstaat, bis zu 100 auf einmal, mit oder ohne Formatierung. Für Softwaretests und Entwicklung.",

  num: "Nur Ziffern",
  multiple_label: "Mehrere CPFs erzeugen",
  select_state: "Bundesstaat auswählen",
  all_states: "Alle Bundesstaaten",
  region_from: "Steuerregion",
  bt: "CPF erzeugen",
  bt_multiple: "CPFs erzeugen",
  cpf_label: "Erzeugter CPF",
  quantity: "Anzahl",
  generated_cpfs: "Erzeugte CPFs ({count})",
  copy_all: "Alle kopieren",
  placeholder: "Klicken Sie auf CPF erzeugen, um eine Nummer zu erstellen",
  warning: "Die erzeugten Nummern sind fiktiv und ausschließlich für Testzwecke bestimmt.",

  d1: "Dieses Werkzeug erzeugt zufällige, mathematisch gültige brasilianische CPF-Nummern, deren Prüfziffern nach dem offiziellen Algorithmus berechnet werden. Die Nummern eignen sich zum Testen von Formularen, zum Befüllen von Staging-Datenbanken und zur Validierung von Geschäftsregeln in Anwendungen — überall dort, wo eine CPF benötigt wird, die die algorithmische Prüfung besteht, aber nicht mit einer realen Person verknüpft ist. Das Werkzeug läuft direkt im Browser, ohne Anmeldung, und ermöglicht die Filterung nach Herkunftsbundesstaat oder die Erzeugung von bis zu 100 Nummern in einem Durchgang.",

  features_title: "Funktionen",
  f_1: "Erzeugung gültiger CPFs nach brasilianischem Bundesstaat",
  f_2: "Stapel von bis zu 100 eindeutigen CPFs",
  f_3: "Formatiert mit Punkten oder reine Ziffern",
  f_4: "Prüfziffern nach Modulo-11-Verfahren berechnet",

  how_title: "Funktionsweise des CPF-Algorithmus",
  how_desc: "Eine CPF besteht aus 11 Ziffern: 9 zur Identifikation und 2 Prüfziffern am Ende. Die ersten 9 Ziffern sind innerhalb jeder Steuerregion fortlaufend vergeben, wobei die neunte Ziffer den Bundesstaat angibt, in dem das Dokument ausgestellt wurde. Die letzten beiden ergeben sich aus einer Modulo-11-Berechnung über die vorhergehenden Ziffern: Jede Ziffer wird mit einem Gewicht multipliziert, die Ergebnisse werden addiert, durch 11 geteilt, und der Rest bestimmt die Prüfziffer. Das Verfahren wurde entwickelt, um Tippfehler zu erkennen. Eine veränderte Ziffer oder zwei vertauschte Ziffern führen fast immer zu einer ungültigen Nummer, sodass jedes System fehlerhafte Eingaben ablehnen kann, ohne die Steuerbehörde zu kontaktieren.",

  use_cases_title: "Anwendungsfälle",
  use_cases_intro: "Wo eine erzeugte CPF im Entwickleralltag sinnvoll ist:",
  use_1_t: "Formularvalidierung",
  use_1_d: "Eingabemasken, Validatoren und Fehlermeldungen in Registrierungsfeldern testen.",
  use_2_t: "Automatisierte Tests",
  use_2_d: "Testszenarien mit Daten befüllen, die die Validierung bestehen, ohne echte CPFs preiszugeben.",
  use_3_t: "Entwicklungsumgebungen",
  use_3_d: "Datensätze in Staging- und Abnahmeumgebungen anlegen, ohne Datenschutzgesetze zu verletzen oder Produktionsdaten zu kopieren.",
  use_4_t: "API-Integration",
  use_4_d: "Endpunkte und Drittanbieterdienste testen, die eine CPF als Parameter erwarten.",
  use_5_t: "Mockups und Demos",
  use_5_d: "Prototypen, Demobildschirme und Präsentationsmaterialien mit glaubwürdigen, aber fiktiven Daten füllen.",

  how_to_use_title: "Anwendung",
  step_1_title: "Eine einzelne CPF erzeugen",
  step_1_desc: "Beim Laden der Seite ist bereits eine erste CPF verfügbar. Für eine weitere klicken Sie auf „CPF erzeugen“ — die Nummer lässt sich mit einem Klick kopieren.",
  step_2_title: "Mehrere Nummern auf einmal erzeugen",
  step_2_desc: "Aktivieren Sie „Mehrere CPFs erzeugen“, wählen Sie die gewünschte Anzahl (zwischen 2 und 100) und klicken Sie auf „CPFs erzeugen“. Die Schaltfläche „Alle kopieren“ überträgt die gesamte Liste in die Zwischenablage.",
  step_3_title: "Format wählen",
  step_3_desc: "Standardmäßig wird die CPF mit Punkten und Bindestrich ausgegeben. Wenn Sie die Nummer ohne Formatierung benötigen (Datenbanken, JSON usw.), aktivieren Sie vor dem Erzeugen die Option „Nur Ziffern“.",

  region0: "Rio Grande do Sul (RS)",
  region1: "Bundesdistrikt (DF), Goiás (GO), Mato Grosso (MT), Mato Grosso do Sul (MS), Tocantins (TO)",
  region2: "Acre (AC), Amapá (AP), Amazonas (AM), Pará (PA), Rondônia (RO), Roraima (RR)",
  region3: "Ceará (CE), Maranhão (MA), Piauí (PI)",
  region4: "Alagoas (AL), Paraíba (PB), Pernambuco (PE), Rio Grande do Norte (RN)",
  region5: "Bahia (BA), Sergipe (SE)",
  region6: "Minas Gerais (MG)",
  region7: "Espírito Santo (ES), Rio de Janeiro (RJ)",
  region8: "São Paulo (SP)",
  region9: "Paraná (PR), Santa Catarina (SC)",

  faq_title: "Häufig gestellte Fragen",
  faq_1_q: "Sind die erzeugten CPFs echt oder bei der brasilianischen Steuerbehörde registriert?",
  faq_1_a: "Nein. Es handelt sich um fiktive Nummern, die mit demselben Validierungsalgorithmus wie bei der Receita Federal erzeugt werden, jedoch ohne jeden Bezug zu realen Personen. Sie bestehen die Prüfung der Kontrollziffern, weil die Berechnung korrekt ist — mehr nicht. Sie sollten keinesfalls für offizielle Zwecke verwendet werden, da sie dort nicht akzeptiert werden und ein solcher Versuch nach brasilianischem Recht eine Straftat darstellen kann.",
  faq_2_q: "Warum gibt es ein Bundesstaat-Feld im Generator?",
  faq_2_a: "Die neunte Ziffer der CPF (direkt vor den Prüfziffern) entspricht der Steuerregion, in der das Dokument ursprünglich ausgestellt wurde. São Paulo entspricht der 8, Rio Grande do Sul der 0 und so weiter. Für bestimmte Testszenarien ist es relevant, Nummern zu erzeugen, die zu einer bestimmten Region passen — dafür ist der Filter gedacht. Andernfalls bleibt „Alle Bundesstaaten“ ausgewählt, und die Region wird zufällig vergeben.",
  faq_3_q: "Wann ist das formatierte Format zu wählen, wann nur Ziffern?",
  faq_3_a: "Das hängt vom Verwendungszweck ab. Formulare und Oberflächen, die dem Endnutzer angezeigt werden, verwenden üblicherweise das formatierte Format (999.999.999-99). Datenbanken, APIs und Backend-Validatoren speichern die Nummer dagegen meist als reine Ziffernfolge ohne Sonderzeichen. Die Option „Nur Ziffern“ erledigt das direkt, ohne dass die Zeichenkette nachträglich aufbereitet werden muss.",
  faq_4_q: "Ist die Nutzung dieses Generators legal?",
  faq_4_a: "Für Softwaretests, Lernzwecke, Demonstrationen und Entwicklungsarbeit: ja — dafür wurde er geschaffen. Was nicht erlaubt ist, ist die Verwendung einer erzeugten CPF in offiziellen Registrierungen, Verträgen, Erklärungen oder anderen Vorgängen, die eine echte Identifikation erfordern. Ein solcher Gebrauch erfüllt nach brasilianischem Recht den Tatbestand der ideologischen Falschbeurkundung (Art. 299 Strafgesetzbuch) und kann je nach Sachverhalt auch als Betrug gewertet werden.",
  faq_5_q: "Was unterscheidet „CPF-Generator“, „Fake-CPF“ und „zufällige CPF“?",
  faq_5_a: "In der Praxis ist damit dasselbe gemeint: eine Nummer, die den mathematischen Regeln einer CPF folgt, aber niemandem zugeordnet ist. Die Begriffe „Fake“ und „fiktiv“ werden in Suchanfragen austauschbar verwendet, in technischen Kontexten bevorzugen Entwickler eher Bezeichnungen wie „Test-CPF“ oder „fiktive CPF“. Das Ergebnis und die zulässigen Anwendungsfälle bleiben identisch.",

  see1: "CNPJ-Generator",
  see2: "Andere Buchstaben",
  see3: "Kreditkartengenerator",
  see4: "Geburtsdatum-Generator"
}
}
</i18n>