<script setup lang="ts">
const { t, locale } = useI18n({ useScope: 'local' })

const COPY_FEEDBACK_MS = 2000
const MAX_RECENT = 10

interface EmojiItem { char: string; name: string; group: string }

const emojiCategories = [
  { icon: '😀', name: 'cat_smileys',    desc: 'cat_smileys_d'    },
  { icon: '👋', name: 'cat_gestures',   desc: 'cat_gestures_d'   },
  { icon: '🧑', name: 'cat_people',     desc: 'cat_people_d'     },
  { icon: '🐶', name: 'cat_animals',    desc: 'cat_animals_d'    },
  { icon: '🍔', name: 'cat_food',       desc: 'cat_food_d'       },
  { icon: '✈️', name: 'cat_travel',     desc: 'cat_travel_d'     },
  { icon: '⚽', name: 'cat_activities', desc: 'cat_activities_d' },
  { icon: '💡', name: 'cat_objects',    desc: 'cat_objects_d'    },
  { icon: '🔣', name: 'cat_symbols',    desc: 'cat_symbols_d'    },
  { icon: '🏳️', name: 'cat_flags',     desc: 'cat_flags_d'      }
]

usePageJsonLd({
  name: t('title'),
  description: t('meta'),
  features: [t('f_1'), t('f_2'), t('f_3'), t('f_4')],
  howToName: t('how_it_works_title'),
  howToSteps: [
    { name: t('hiw_1_title'), text: t('hiw_1_desc') },
    { name: t('hiw_2_title'), text: t('hiw_2_desc') },
    { name: t('hiw_3_title'), text: t('hiw_3_desc') }
  ],
  faq: [
    { question: t('faq_1_q'), answer: t('faq_1_a') },
    { question: t('faq_2_q'), answer: t('faq_2_a') },
    { question: t('faq_3_q'), answer: t('faq_3_a') },
    { question: t('faq_4_q'), answer: t('faq_4_a') },
    { question: t('faq_5_q'), answer: t('faq_5_a') }
  ]
})

useSeoMeta({
  title: t('m_title'),
  description: t('meta'),
  ogTitle: t('m_title'),
  ogDescription: t('meta'),
  ogType: 'website',
  ogLocale: locale.value === 'pt' ? 'pt_BR' : locale.value,
  twitterCard: 'summary_large_image',
  twitterTitle: t('m_title'),
  twitterDescription: t('meta')
})

const state = reactive({
  keyword: '',
  tab: 0,
  loading: true,
  data: [] as EmojiItem[],
  emojis: [] as EmojiItem[],
  categories: [] as string[],
  recentlyCopied: [] as string[],
  copiedChar: null as string | null,
  copyAnnouncement: ''
})

onMounted(async () => {
  try {
    const saved = localStorage.getItem('recentEmojis')
    if (saved) state.recentlyCopied = JSON.parse(saved).slice(0, MAX_RECENT)
  } catch {}

  try {
    const res = await fetch('https://unpkg.com/emoji.json')
    const json = await res.json()
    const seen = new Set<string>()
    const groups = new Set<string>()
    const data: EmojiItem[] = []

    for (const item of json) {
      if (!seen.has(item.name)) {
        seen.add(item.name)
        groups.add(item.group)
        data.push({ char: item.char, name: item.name.toLowerCase(), group: item.group })
      }
    }

    state.data = data
    state.categories = [...groups]
    filter()
  } catch {}

  state.loading = false
})

watch(() => state.tab, filter)
watch(() => state.keyword, filter)

function filter() {
  if (state.keyword.length >= 2) {
    const kw = state.keyword.toLowerCase()
    state.emojis = state.data.filter(item => item.name.includes(kw))
  } else {
    state.emojis = state.data.filter(item => item.group === state.categories[state.tab])
  }
}

async function copy(char: string) {
  try {
    await navigator.clipboard.writeText(char)
    state.copiedChar = char
    state.copyAnnouncement = t('copied_announce')
    addToRecent(char)
    setTimeout(() => {
      state.copiedChar = null
      state.copyAnnouncement = ''
    }, COPY_FEEDBACK_MS)
  } catch {}
}

function addToRecent(emoji: string) {
  const filtered = state.recentlyCopied.filter(e => e !== emoji)
  state.recentlyCopied = [emoji, ...filtered].slice(0, MAX_RECENT)
  try {
    localStorage.setItem('recentEmojis', JSON.stringify(state.recentlyCopied))
  } catch {}
}

defineI18nRoute({
  paths: {
    en: '/emoji-picker',
    pt: '/seletor-de-emojis',
    es: '/selector-de-emojis',
    fr: '/selecteur-d-emojis',
    it: '/selettore-di-emoji',
    id: '/pemilih-emoji'
  }
})
</script>

<template>
  <ToolPage
    :title="t('title')"
    :description="t('meta')"
    :show-ads="state.emojis.length > 0"
    :see-also-links="[
      { label: t('see1'), to: 'symbol-picker' },
      { label: t('see2'), to: 'barcode-generator' },
      { label: t('see3'), to: 'random-colors' },
      { label: t('see4'), to: 'dominant-colors-of-the-image' }
    ]"
  >
    <div class="space-y-5 animate-in fade-in slide-in-from-bottom-4 duration-500">

      <!-- Campo de busca -->
      <div class="form-control w-full">
        <label for="emoji-search" class="label">
          <span class="label-text font-bold text-base-content">{{ t('search_label') }}</span>
          <span v-if="state.keyword && !state.loading" class="label-text-alt text-base-content/70 text-sm">
            {{ state.emojis.length }} {{ t('found') }}
          </span>
        </label>
        <div class="relative">
          <input
            id="emoji-search"
            v-model="state.keyword"
            type="search"
            class="input input-bordered input-lg w-full bg-base-200 focus:bg-base-200 transition-all rounded-2xl pl-12 pr-12"
            :placeholder="t('plc')"
            enterkeyhint="search"
            autocomplete="off"
            spellcheck="false"
            autofocus
          />
          <Icon
            name="material-symbols:search-rounded"
            class="absolute left-4 top-1/2 -translate-y-1/2 w-5 h-5 text-base-content/40 pointer-events-none"
            aria-hidden="true"
          />
          <button
            v-if="state.keyword"
            type="button"
            @click="state.keyword = ''"
            :aria-label="t('clear')"
            class="absolute right-3 top-1/2 -translate-y-1/2 btn btn-circle btn-ghost btn-xs text-base-content/40 hover:text-base-content"
          >
            <Icon name="material-symbols:close-rounded" class="w-4 h-4" aria-hidden="true" />
          </button>
        </div>
      </div>

      <!-- Emojis copiados recentemente -->
      <div v-if="state.recentlyCopied.length > 0" class="bg-base-200 rounded-2xl px-4 py-3">
        <p class="text-sm font-semibold text-base-content/60 uppercase tracking-wide mb-2">{{ t('recently_copied') }}</p>
        <div class="flex flex-wrap gap-1">
          <button
            v-for="(emoji, i) in state.recentlyCopied"
            :key="i"
            type="button"
            @click="copy(emoji)"
            class="text-3xl p-1.5 rounded-xl transition-all duration-150 hover:bg-base-300 hover:scale-110 active:scale-95"
            :class="state.copiedChar === emoji ? 'bg-success/20 scale-110' : ''"
          >{{ emoji }}</button>
        </div>
      </div>

      <!-- Abas de categorias -->
      <div v-show="!state.keyword">
        <p class="text-sm font-semibold text-base-content mb-2">{{ t('categories') }}</p>
        <div class="flex flex-wrap gap-1.5">
          <button
            v-for="(cat, i) in emojiCategories"
            :key="i"
            type="button"
            @click="state.tab = i"
            :title="t(cat.name)"
            :aria-label="t(cat.name)"
            :aria-pressed="state.tab === i"
            class="btn btn-sm rounded-xl"
            :class="state.tab === i ? 'btn-primary' : 'btn-ghost bg-base-200'"
          >
            <span class="text-xl leading-none">{{ cat.icon }}</span>
          </button>
        </div>
        <p class="text-sm text-primary mt-2 font-medium">{{ t(emojiCategories[state.tab].name) }}</p>
      </div>

      <!-- Grade de emojis (altura fixa para evitar CLS) -->
      <div
        class="bg-base-100 rounded-2xl border border-base-content/10 shadow-sm overflow-y-auto p-3"
        style="height: 420px"
        :aria-label="t('results_label')"
        role="list"
      >
        <!-- Loading -->
        <div v-if="state.loading" class="flex items-center justify-center h-full">
          <span class="loading loading-spinner loading-lg text-primary"></span>
        </div>

        <!-- Estado vazio na busca -->
        <div
          v-else-if="state.emojis.length === 0 && state.keyword"
          class="flex flex-col items-center justify-center h-full gap-2"
        >
          <span class="text-4xl">🔍</span>
          <p class="text-base-content/60 text-sm">{{ t('empty') }}</p>
        </div>

        <!-- Grade -->
        <div v-else class="flex flex-wrap gap-0.5">
          <button
            v-for="item in state.emojis"
            :key="item.char"
            type="button"
            role="listitem"
            :title="item.name"
            :aria-label="item.name"
            @click="copy(item.char)"
            class="text-3xl p-2 rounded-xl transition-all duration-150 hover:bg-base-200 hover:scale-110 active:scale-95"
            :class="state.copiedChar === item.char ? 'bg-success/20 scale-110' : ''"
          >{{ item.char }}</button>
        </div>
      </div>

      <!-- Feedback de cópia -->
      <Transition
        enter-active-class="transition duration-200 ease-out"
        enter-from-class="opacity-0 translate-y-1"
        enter-to-class="opacity-100 translate-y-0"
        leave-active-class="transition duration-150 ease-in"
        leave-from-class="opacity-100 translate-y-0"
        leave-to-class="opacity-0 translate-y-1"
      >
        <div
          v-if="state.copiedChar"
          class="flex items-center gap-2 bg-success/10 border border-success/20 text-success rounded-2xl px-4 py-2.5 text-sm font-medium w-fit"
        >
          <Icon name="material-symbols:check-rounded" class="w-4 h-4 shrink-0" aria-hidden="true" />
          <span>{{ state.copiedChar }} {{ t('copied_label') }}</span>
        </div>
      </Transition>

      <!-- Anúncio acessível para leitores de tela -->
      <div role="status" aria-live="polite" aria-atomic="true" class="sr-only">
        {{ state.copyAnnouncement }}
      </div>

    </div>

    <template #info>
      <div class="space-y-8">

        <p>{{ t('d1') }}</p>

        <UseCaseSection
          :title="t('use_cases_title')"
          :items="[
            { title: t('uc_1_title'), description: t('uc_1_desc') },
            { title: t('uc_2_title'), description: t('uc_2_desc') },
            { title: t('uc_3_title'), description: t('uc_3_desc') },
            { title: t('uc_4_title'), description: t('uc_4_desc') }
          ]"
        />

        <HowToSection
          :title="t('how_it_works_title')"
          :items="[
            { title: t('hiw_1_title'), description: t('hiw_1_desc') },
            { title: t('hiw_2_title'), description: t('hiw_2_desc') },
            { title: t('hiw_3_title'), description: t('hiw_3_desc') }
          ]"
        />

        <!-- Glossário de categorias -->
        <div>
          <h2 class="text-xl font-bold mb-4">{{ t('categories_title') }}</h2>
          <p class="text-base-content mb-4">{{ t('categories_intro') }}</p>
          <div class="grid gap-3 sm:grid-cols-2">
            <div
              v-for="cat in emojiCategories"
              :key="cat.name"
              class="flex gap-2"
            >
              <span class="shrink-0 mt-0.5 text-lg leading-none" aria-hidden="true">{{ cat.icon }}</span>
              <p>
                <strong>{{ t(cat.name) }}:</strong>
                {{ t(cat.desc) }}
              </p>
            </div>
          </div>
        </div>

        <!-- Seção educacional sobre emojis e Unicode -->
        <div>
          <h2 class="text-xl font-bold mb-3">{{ t('unicode_title') }}</h2>
          <p class="text-base-content leading-relaxed">{{ t('unicode_p1') }}</p>
          <p class="text-base-content leading-relaxed mt-2">{{ t('unicode_p2') }}</p>
        </div>

        <FaqSection
          :title="t('faq_title')"
          :items="[
            { question: t('faq_1_q'), answer: t('faq_1_a') },
            { question: t('faq_2_q'), answer: t('faq_2_a') },
            { question: t('faq_3_q'), answer: t('faq_3_a') },
            { question: t('faq_4_q'), answer: t('faq_4_a') },
            { question: t('faq_5_q'), answer: t('faq_5_a') }
          ]"
        />

      </div>
    </template>
  </ToolPage>
</template>

<i18n lang="yaml">
{
  pt: {
    m_title: "Copiar e Colar Emojis - Seletor de Emojis Online 😀❤️🎉",
    title: "Copiar e Colar Emojis",
    meta: "Seletor de emojis para copiar e colar. Encontre emojis por palavra-chave ou explore categorias de emojis como sorrisos, animais, comida e símbolos.",
    d1: "Encontre o emoji ideal para copiar e colar em mensagens do WhatsApp, biografias do Instagram ou nicks de jogos. Busque emojis por palavra-chave ou navegue por categorias como rostos, animais, comida e símbolos. Esta ferramenta organiza os emojis para facilitar a busca e permitir cópia rápida em qualquer dispositivo.",

    search_label: "Pesquisar emojis",
    plc: "Pesquisar... (ex: heart, fire, smile)",
    clear: "Limpar busca",
    categories: "Categorias",
    recently_copied: "Copiados recentemente",
    copied_label: "Copiado!",
    copied_announce: "Emoji copiado para a área de transferência",
    found: "encontrados",
    empty: "Nenhum emoji encontrado. Tente outra palavra-chave em inglês.",
    results_label: "Grade de emojis",

    use_cases_title: "Onde usar seus emojis",
    uc_1_title: "Instagram, TikTok e redes sociais",
    uc_1_desc: "Dê vida à sua bio, legendas e comentários com emojis expressivos. Posts com emojis geram mais engajamento e tornam o perfil mais humano e atraente.",
    uc_2_title: "WhatsApp e Telegram",
    uc_2_desc: "Expresse emoções com mais precisão nas conversas. Um emoji certo comunica o tom da mensagem sem espaço para mal-entendidos e torna os textos mais rápidos de ler.",
    uc_3_title: "Discord e comunidades online",
    uc_3_desc: "Personalize seu nome de usuário, reaja a mensagens e torne os canais mais dinâmicos. Emojis são a linguagem natural das comunidades digitais.",
    uc_4_title: "E-mails, apresentações e conteúdo",
    uc_4_desc: "Use com moderação para destacar pontos importantes, humanizar títulos e chamar atenção em assuntos de e-mail. Um emoji bem colocado aumenta a taxa de abertura.",

    how_it_works_title: "Como usar",
    hiw_1_title: "Navegue por categorias ou pesquise",
    hiw_1_desc: "Use as abas para explorar emojis por tema, ou digite uma palavra-chave em inglês (ex: heart, star, fire) para encontrar o emoji exato que você precisa.",
    hiw_2_title: "Clique para copiar",
    hiw_2_desc: "Um clique em qualquer emoji copia automaticamente para a área de transferência. Não é preciso selecionar, arrastar ou pressionar nenhuma tecla extra.",
    hiw_3_title: "Cole onde quiser",
    hiw_3_desc: "Uma vez copiado, cole onde quiser. Funciona em qualquer plataforma que aceite texto.",

    categories_title: "Categorias de emojis disponíveis",
    categories_intro: "Os emojis estão organizados em 10 categorias para facilitar a navegação. Veja o que cada uma inclui:",
    cat_smileys: "Sorrisos e Emoções",
    cat_smileys_d: "Carinhas felizes, tristes, surpresas, apaixonadas e muito mais. É a categoria mais usada para expressar sentimentos de forma rápida e universal.",
    cat_gestures: "Gestos",
    cat_gestures_d: "Mãos acenando, polegares, palmas e gestos do cotidiano. Ideais para expressar concordância, cumprimentos, pedidos e reações rápidas.",
    cat_people: "Pessoas",
    cat_people_d: "Avatares, famílias, profissões e representações diversas. Úteis para humanizar mensagens e representar diferentes contextos e identidades.",
    cat_animals: "Animais e Natureza",
    cat_animals_d: "Pets, animais selvagens, plantas, flores e fenômenos naturais. Perfeitos para quem ama o mundo natural ou quer dar um toque orgânico à mensagem.",
    cat_food: "Comida e Bebida",
    cat_food_d: "Frutas, refeições, sobremesas, bebidas e utensílios de cozinha. Muito usados em receitas, posts de gastronomia e conversas sobre comida.",
    cat_travel: "Viagens e Lugares",
    cat_travel_d: "Meios de transporte, pontos turísticos, construções e ambientes. Ótimos para posts de viagem, check-ins e conversas sobre destinos.",
    cat_activities: "Atividades",
    cat_activities_d: "Esportes, jogos, artes, música e hobbies. Expressam paixões, conquistas e momentos de lazer com clareza e dinamismo.",
    cat_objects: "Objetos",
    cat_objects_d: "Tecnologia, ferramentas, roupas, acessórios e objetos do dia a dia. Comunicam contextos, situações e temas específicos de forma visual.",
    cat_symbols: "Símbolos",
    cat_symbols_d: "Sinais de trânsito, símbolos religiosos, moedas, marcas e ícones. Úteis para contextualizar temas específicos e adicionar significados visuais.",
    cat_flags: "Bandeiras",
    cat_flags_d: "Bandeiras de países, territórios e comunidades. Excelentes para indicar origem, apoio a um time, contexto geográfico ou identidade cultural.",

    unicode_title: "Por que emojis funcionam em qualquer plataforma",
    unicode_p1: "Emojis são caracteres Unicode padronizados e não imagens ou arquivos. Isso significa que funcionam em todos os dispositivos, sistemas operacionais e aplicativos que suportam texto: do Android ao iPhone, do Windows ao Mac, do Instagram ao Discord. Quando você copia um emoji aqui e cola em outro lugar, o resultado é sempre compatível.",
    unicode_p2: "A aparência visual pode variar levemente entre plataformas (o ❤️ no iOS tem um estilo diferente do Android), mas o significado e a compatibilidade são universais.",

    faq_title: "Perguntas e Respostas",
    faq_1_q: "Como copiar e colar emojis pelo computador?",
    faq_1_a: "Acesse esta página, clique em qualquer emoji para copiá-lo automaticamente para a área de transferência. Em seguida, vá ao aplicativo ou site desejado e cole. Não é necessário instalar nada.",
    faq_2_q: "Os emojis funcionam na bio e nas legendas do Instagram?",
    faq_2_a: "Sim. O Instagram aceita todos os emojis Unicode na bio, legendas, comentários, histórias e nomes de usuário. Basta copiar o emoji nesta página e colar diretamente no Instagram, seja pelo celular ou pelo computador.",
    faq_3_q: "Posso usar emojis no nome e nas mensagens do WhatsApp?",
    faq_3_a: "Sim. O WhatsApp suporta emojis Unicode no nome do perfil, nas mensagens, nos status e nas descrições de grupos.",
    faq_4_q: "Emojis funcionam em nicknames de jogos como Free Fire e Roblox?",
    faq_4_a: "Depende do jogo. Alguns aceitam emojis nos nicknames, outros bloqueiam caracteres Unicode especiais por questões de moderação. Emojis simples como ❤️ e ⭐ costumam ter melhor compatibilidade. Sempre teste antes de salvar o nick.",
    faq_5_q: "Como funciona o filtro de busca?",
    faq_5_a: "À medida que você digita a palavra-chave, a ferramenta filtra automaticamente a lista, exibindo apenas os emojis que correspondem ao termo pesquisado. Isso permite encontrar o ícone exato de forma rápida e dinâmica.",

    see1: "Seletor de Símbolos",
    see2: "Gerador de Código de Barras",
    see3: "Cores Aleatórias",
    see4: "Cores Dominantes da Imagem",

    f_1: "Mais de 3.000 emojis disponíveis",
    f_2: "Cópia com um clique",
    f_3: "Pesquisa por palavra-chave",
    f_4: "Histórico de emojis recentes"
  },
  en: {
    cat_smileys: "Smileys & Emotion",
    cat_gestures: "Gestures",
    cat_people: "People",
    cat_animals: "Animals & Nature",
    cat_food: "Food & Drink",
    cat_travel: "Travel & Places",
    cat_activities: "Activities",
    cat_objects: "Objects",
    cat_symbols: "Symbols",
    cat_flags: "Flags",
    see1: "Symbol Picker",
    see2: "Barcode Generator",
    see3: "Random Colors",
    see4: "Dominant Colors of the Image"
  },
  es: {
    cat_smileys: "Emoticonos y Emociones",
    cat_gestures: "Gestos",
    cat_people: "Personas",
    cat_animals: "Animales y Naturaleza",
    cat_food: "Comida y Bebida",
    cat_travel: "Viajes y Lugares",
    cat_activities: "Actividades",
    cat_objects: "Objetos",
    cat_symbols: "Símbolos",
    cat_flags: "Banderas",
    see1: "Selector de Símbolos",
    see2: "Generador de Código de Barras",
    see3: "Colores Aleatorios",
    see4: "Colores Dominantes de la Imagen"
  },
  fr: {
    cat_smileys: "Smileys et Émotions",
    cat_gestures: "Gestes",
    cat_people: "Personnes",
    cat_animals: "Animaux et Nature",
    cat_food: "Nourriture et Boisson",
    cat_travel: "Voyages et Lieux",
    cat_activities: "Activités",
    cat_objects: "Objets",
    cat_symbols: "Symboles",
    cat_flags: "Drapeaux",
    see1: "Sélecteur de Symboles",
    see2: "Générateur de Code-Barres",
    see3: "Couleurs Aléatoires",
    see4: "Couleurs Dominantes de l'Image"
  },
  it: {
    cat_smileys: "Faccine ed Emozioni",
    cat_gestures: "Gesti",
    cat_people: "Persone",
    cat_animals: "Animali e Natura",
    cat_food: "Cibo e Bevande",
    cat_travel: "Viaggi e Luoghi",
    cat_activities: "Attività",
    cat_objects: "Oggetti",
    cat_symbols: "Simboli",
    cat_flags: "Bandiere",
    see1: "Selettore di Simboli",
    see2: "Generatore di Codice a Barre",
    see3: "Colori Casuali",
    see4: "Colori Dominanti dell'Immagine"
  },
  id: {
    cat_smileys: "Smiley & Emosi",
    cat_gestures: "Gestur",
    cat_people: "Orang",
    cat_animals: "Hewan & Alam",
    cat_food: "Makanan & Minuman",
    cat_travel: "Perjalanan & Tempat",
    cat_activities: "Aktivitas",
    cat_objects: "Benda",
    cat_symbols: "Simbol",
    cat_flags: "Bendera",
    see1: "Pemilih Simbol",
    see2: "Generator Barcode",
    see3: "Warna Acak",
    see4: "Warna Dominan Gambar"
  }
}
</i18n>
