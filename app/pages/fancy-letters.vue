<script setup lang="ts">
import { dfs, f1, f2, f3, f4, f5, d1 } from '~/assets/js/fonts.js'

const { t, locale } = useI18n({ useScope: 'local' })

const MAX_INPUT_LENGTH = 60
const COPY_FEEDBACK_MS = 2000
const COMBINING_DIACRITICS = /[\u0300-\u036f]/g

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
    { question: t('faq_5_q'), answer: t('faq_5_a') },
    { question: t('faq_6_q'), answer: t('faq_6_a') },
    { question: t('faq_7_q'), answer: t('faq_7_a') }
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

function toArray(text: string): string[] {
  const arr: string[] = []
  for (const c of text) arr.push(c)
  if (arr.length === 26) arr.push(...arr.slice())
  return arr
}

const allFonts: string[][] = []
for (const font of f1) allFonts.push(toArray(font))
for (const font of f2) allFonts.push(font.split(' '))

const fontStyles = [
  { name: 'st_cursive', desc: 'st_cursive_d' },
  { name: 'st_gothic',  desc: 'st_gothic_d'  },
  { name: 'st_bubble',  desc: 'st_bubble_d'  },
  { name: 'st_square',  desc: 'st_square_d'  },
  { name: 'st_mono',    desc: 'st_mono_d'    },
  { name: 'st_invert',  desc: 'st_invert_d'  },
  { name: 'st_manga',   desc: 'st_manga_d'   },
  { name: 'st_zalgo',   desc: 'st_zalgo_d'   }
]

const state = reactive({
  text: '',
  letters: [] as string[],
  decorations: d1.replaceAll('t', t('text')).split('&') as string[],
  selectedDecoration: null as string | null,
  copiedIndex: null as number | null,
  copyAnnouncement: ''
})

function convert(text: string) {
  const raw = text.trim()
  const src = raw === '' ? t('preview_text') : raw
  const noAccents = src.normalize('NFD').replace(COMBINING_DIACRITICS, '')
  const letters: string[] = []

  for (const font of allFonts) {
    let out = ''
    for (const c of noAccents) {
      const idx = dfs.indexOf(c)
      out += idx > -1 && idx < font.length ? font[idx] : c
    }
    letters.push(out)
  }

  for (const symbol of f4) {
    let out = ''
    for (const c of src) out += c.replace(/([^\s])/g, symbol)
    letters.push(out)
  }

  for (const symbol of f3) letters.push(src.split('').join(symbol) + symbol)
  for (const symbol of f5) letters.push(symbol + src.split('').join(symbol) + symbol)

  if (state.selectedDecoration) {
    for (let i = 0; i < letters.length; i++) {
      letters[i] = state.selectedDecoration.replace(t('text'), letters[i])
    }
  }

  state.letters = letters
}

convert('')

watch(() => state.text, (val) => {
  state.copiedIndex = null
  convert(val)
})

watch(() => state.selectedDecoration, () => {
  state.copiedIndex = null
  convert(state.text)
})

function clearInput() {
  state.text = ''
}

async function copy(text: string, index: number) {
  try {
    await navigator.clipboard.writeText(text)
    state.copiedIndex = index
    state.copyAnnouncement = t('copied_announce')
    setTimeout(() => {
      state.copiedIndex = null
      state.copyAnnouncement = ''
    }, COPY_FEEDBACK_MS)
  } catch {
    // Fallback silencioso — navegadores antigos sem Clipboard API
  }
}

defineI18nRoute({
  paths: {
    en: '/fancy-letters',
    pt: '/letras-diferentes',
    es: '/letras-diferentes',
    fr: '/lettres-differentes',
    it: '/lettere-diverse',
    id: '/huruf-berbeda'
  }
})
</script>

<template>
  <ToolPage
    :title="t('title')"
    :description="t('meta')"
    :show-ads="state.text.length > 0"
    :see-also-links="[
      { label: t('see1'), to: 'url-encoder-decoder' },
      { label: t('see2'), to: 'password-generator' },
      { label: t('see3'), to: 'list-randomizer' },
      { label: t('see4'), to: 'email-extractor' }
    ]"
  >
    <div class="space-y-5 animate-in fade-in slide-in-from-bottom-4 duration-500">

      <!-- Input principal com contador e botão limpar -->
      <div class="form-control w-full">
        <label for="fancy-input" class="label">
          <span class="label-text font-bold text-base-content">{{ t('in_label') }}</span>
          <span class="label-text-alt text-base-content/50 text-sm">
            {{ state.text.length }} / {{ MAX_INPUT_LENGTH }}
          </span>
        </label>
        <div class="relative">
          <input
            id="fancy-input"
            v-model="state.text"
            type="text"
            :maxlength="MAX_INPUT_LENGTH"
            class="input input-bordered input-lg w-full bg-base-200 focus:bg-base-200 transition-all rounded-2xl pr-12"
            :placeholder="t('plc')"
            enterkeyhint="done"
            autocomplete="off"
            spellcheck="false"
            autofocus
          />
          <button
            v-if="state.text.length > 0"
            type="button"
            @click="clearInput"
            :aria-label="t('clear')"
            class="absolute right-3 top-1/2 -translate-y-1/2 btn btn-circle btn-ghost btn-xs text-base-content/40 hover:text-base-content"
          >
            <Icon name="material-symbols:close-rounded" class="w-4 h-4" aria-hidden="true" />
          </button>
        </div>
      </div>

      <!-- Seletor de decoração -->
      <div class="form-control w-full sm:w-fit">
        <label for="decoration-select" class="label">
          <span class="label-text font-semibold text-base-content">{{ t('decoration') }}</span>
        </label>
        <select
          id="decoration-select"
          v-model="state.selectedDecoration"
          class="select select-bordered bg-base-200 rounded-2xl w-full sm:w-auto ml-2"
        >
          <option :value="null">{{ t('no_deco') }}</option>
          <option v-for="dec in state.decorations" :key="dec" :value="dec">{{ dec }}</option>
        </select>
      </div>

      <!-- Lista de resultados (altura fixa para evitar CLS) -->
      <ul
        class="bg-base-100 rounded-2xl border border-base-content/10 shadow-sm overflow-y-auto"
        style="height: 420px"
        :aria-label="t('results_label')"
      >
        <li
          v-for="(letter, i) in state.letters"
          :key="i"
          class="flex items-center gap-3 px-4 py-3 hover:bg-base-200/60 transition-colors border-b border-base-content/5 last:border-0"
        >
          <span class="flex-1 break-all leading-relaxed select-all text-xl">{{ letter }}</span>
          <button
            type="button"
            @click="copy(letter, i)"
            :aria-label="t('copy')"
            class="btn btn-square btn-ghost btn-sm shrink-0 transition-all duration-200 active:scale-90"
            :class="state.copiedIndex === i ? 'text-success' : 'text-base-content/40 hover:text-primary'"
          >
            <Transition
              enter-active-class="transition duration-150 ease-out"
              enter-from-class="scale-50 opacity-0"
              enter-to-class="scale-100 opacity-100"
              leave-active-class="transition duration-100 ease-in"
              leave-from-class="scale-100 opacity-100"
              leave-to-class="scale-50 opacity-0"
              mode="out-in"
            >
              <Icon
                :key="state.copiedIndex === i ? 'check' : 'copy'"
                :name="state.copiedIndex === i ? 'material-symbols:check-rounded' : 'material-symbols:content-copy-outline'"
                class="w-5 h-5"
                aria-hidden="true"
              />
            </Transition>
          </button>
        </li>
      </ul>

      <!-- Anúncio acessível para leitores de tela ao copiar -->
      <div
        role="status"
        aria-live="polite"
        aria-atomic="true"
        class="sr-only"
      >
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

        <!-- Glossário de estilos -->
        <div>
          <h2 class="text-xl font-bold mb-4">{{ t('styles_title') }}</h2>
          <p class="text-base-content mb-4">{{ t('styles_intro') }}</p>
          <div class="grid gap-3 sm:grid-cols-2">
            <div
              v-for="style in fontStyles"
              :key="style.name"
              class="flex gap-2"
            >
              <span class="text-primary font-bold shrink-0 mt-0.5" aria-hidden="true">✓</span>
              <p>
                <strong>{{ t(style.name) }}:</strong>
                {{ t(style.desc) }}
              </p>
            </div>
          </div>
        </div>

        <!-- Seção educacional: substitui o keyword stuffing antigo -->
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
            { question: t('faq_5_q'), answer: t('faq_5_a') },
            { question: t('faq_6_q'), answer: t('faq_6_a') },
            { question: t('faq_7_q'), answer: t('faq_7_a') }
          ]"
        />

      </div>
    </template>
  </ToolPage>
</template>

<i18n lang="yaml">
{
  pt: {
    m_title: "Letras Diferentes - Gerador de Letras Bonitas - Copiar e Colar",
    title: "Letras Diferentes",
    meta: "Crie letras diferentes e bonitas para Instagram, WhatsApp, Free Fire e Discord. Mais de 30 estilos de fontes estilizadas prontas para copiar e colar.",
    d1: "Transforme qualquer texto comum em algo único, decorativo e cheio de personalidade. Esta ferramenta gera dezenas de variações ao mesmo tempo, com fontes cursivas, góticas, bolha, monoespaçadas, invertidas e muitas outras, prontas para você usar na bio do Instagram, em nicks de jogos, status do WhatsApp ou em qualquer aplicativo que aceite texto.",
    in_label: "Seu texto",
    plc: "Digite seu nome, frase ou nick aqui...",
    preview_text: "Exemplo",
    text: "Texto",
    decoration: "Símbolos e Bordas:",
    no_deco: "Sem decoração",
    copy: "Copiar",
    clear: "Limpar texto",
    copied_announce: "Texto copiado para a área de transferência",
    results_label: "Lista de estilos gerados",

    use_cases_title: "Onde usar suas letras personalizadas",
    uc_1_title: "Instagram, TikTok e redes sociais",
    uc_1_desc: "Destaque a bio, a legenda ou o nome do seu perfil com fontes decorativas. Como tudo é baseado em símbolos universais, funciona direto, sem instalar nenhum aplicativo extra.",
    uc_2_title: "Nicknames para jogos online",
    uc_2_desc: "Crie nicks únicos para Free Fire, Roblox, Fortnite, Valorant e outros. Combine estilos com bordas decorativas para um visual marcante no seu perfil de jogador.",
    uc_3_title: "WhatsApp, Telegram e Discord",
    uc_3_desc: "Personalize seu nome de exibição, status, recados e mensagens. Os caracteres funcionam em praticamente todos os aplicativos de mensagem modernos.",
    uc_4_title: "E-mails, apresentações e currículos",
    uc_4_desc: "Use com moderação para destacar títulos, citações ou seções específicas em comunicações digitais. Ideal para chamar atenção em assuntos de e-mail e slides.",

    how_it_works_title: "Como usar o gerador de letras bonitas",
    hiw_1_title: "Digite ou cole seu texto",
    hiw_1_desc: "Escreva seu nome, frase ou nick no campo principal. Os resultados aparecem automaticamente em mais de 30 estilos diferentes, sem precisar clicar em gerar.",
    hiw_2_title: "Adicione bordas e símbolos (opcional)",
    hiw_2_desc: "Use o seletor de decoração para envolver o texto com molduras de estrelas, corações, ondas e outros enfeites. Combine estilos até encontrar o visual perfeito.",
    hiw_3_title: "Copie e cole onde quiser",
    hiw_3_desc: "Clique no ícone ao lado do estilo que mais gostou. O texto vai direto para a área de transferência, pronto para colar na sua rede social, jogo ou aplicativo.",

    styles_title: "Estilos de letras disponíveis",
    styles_intro: "A ferramenta cobre as principais categorias de fontes Unicode usadas em redes sociais e jogos. Veja abaixo o que cada uma representa:",
    st_cursive: "Cursiva e Script",
    st_cursive_d: "Imitam uma caligrafia elegante e fluida. Ideais para biografias sofisticadas, nomes artísticos e convites digitais.",
    st_gothic: "Gótica e Fraktur",
    st_gothic_d: "Traços fortes inspirados em manuscritos medievais. Boa escolha para um visual com peso, tradição ou pegada heavy metal.",
    st_bubble: "Bolha (Bubble)",
    st_bubble_d: "Letras arredondadas e divertidas, que parecem bolhas infladas. Funcionam bem em mensagens casuais e perfis lúdicos.",
    st_square: "Quadrada (Square)",
    st_square_d: "Caracteres dentro de molduras quadradas, criando um visual geométrico e moderno muito usado em nicks de jogos.",
    st_mono: "Monoespaçada (Retro)",
    st_mono_d: "Cada caractere ocupa exatamente o mesmo espaço, lembrando máquinas de escrever e terminais. Estética limpa e minimalista.",
    st_invert: "Invertida e Espelhada",
    st_invert_d: "Texto virado de cabeça para baixo ou refletido. Útil para efeitos visuais inusitados e brincadeiras nas redes.",
    st_manga: "Mangá",
    st_manga_d: "Inspirada na tipografia de quadrinhos japoneses, com traços dinâmicos que dão movimento ao texto.",
    st_zalgo: "Zalgo e Glitch",
    st_zalgo_d: "Efeito de texto corrompido, com símbolos sobrepostos formando uma trama caótica. Combina com estéticas de terror, mistério e cyberpunk.",

    unicode_title: "Por que essas letras funcionam em qualquer plataforma",
    unicode_p1: "Apesar de parecerem fontes especiais, essas letras não são fontes no sentido técnico. São caracteres reais do padrão Unicode, o mesmo sistema universal que define os emojis e os alfabetos do mundo todo. É por isso que o resultado aparece igual no Android, iPhone, Windows ou Mac, sem precisar instalar nada.",
    unicode_p2: "Como cada caractere é um símbolo independente, o texto pode ser colado em qualquer campo que aceite escrita comum: bio do Instagram, nome no WhatsApp, recado no Discord, nick em jogos e até em formulários. A única limitação aparece quando o aplicativo bloqueia certos blocos do Unicode por questão de moderação ou segurança.",

    faq_title: "Perguntas frequentes",
    faq_1_q: "Como gerar letras diferentes para copiar e colar?",
    faq_1_a: "Basta digitar o texto desejado no campo no topo da página. A ferramenta converte sua frase instantaneamente em mais de 30 estilos. Depois, clique no botão de copiar ao lado do estilo escolhido e cole onde quiser.",
    faq_2_q: "As letras personalizadas funcionam na bio do Instagram?",
    faq_2_a: "Sim. O Instagram aceita esses caracteres na bio, nos comentários, nas legendas e até nos stories. Como são símbolos Unicode universais, não há necessidade de aplicativos extras.",
    faq_3_q: "Posso usar essas letras no nome do WhatsApp?",
    faq_3_a: "Sim. O WhatsApp suporta esses caracteres no nome de perfil, nas mensagens, nos status e na descrição de grupos. Vale lembrar que algumas combinações podem ficar pequenas em telas menores.",
    faq_4_q: "Funcionam em nicks de Free Fire, Roblox, Fortnite e Valorant?",
    faq_4_a: "Na maioria dos casos, sim. Esses jogos aceitam grande parte dos caracteres Unicode para criar nicks únicos. Alguns símbolos específicos podem ser bloqueados pelas regras internas de cada jogo, então vale testar antes de salvar.",
    faq_5_q: "Por que algumas letras aparecem como quadrados (□) ou pontos de interrogação?",
    faq_5_a: "Isso acontece quando o sistema operacional, navegador ou aplicativo é antigo e não tem a fonte instalada para renderizar aquele caractere específico. Em celulares e computadores atualizados, a maioria aparece sem problema.",
    faq_6_q: "Funciona no Discord, Telegram e Twitter (X)?",
    faq_6_a: "Sim. Discord, Telegram e Twitter aceitam todos os estilos. No Discord você pode usar no nome de exibição, nos canais e em mensagens diretas. Cada plataforma tem suas próprias regras de moderação, mas, em geral, esses caracteres são tratados como texto comum.",
    faq_7_q: "Essas são fontes ou símbolos? Qual a diferença?",
    faq_7_a: "Tecnicamente são símbolos Unicode, não fontes. Uma fonte muda a aparência das mesmas letras (a letra A continua sendo um A). Aqui, cada estilo é um conjunto distinto de caracteres dentro do padrão Unicode. Por isso o resultado se mantém ao copiar para qualquer aplicativo, sem depender da fonte instalada no dispositivo.",

    see1: "Codificador de URL",
    see2: "Gerador de Senhas",
    see3: "Randomizador de Listas",
    see4: "Extrator de E-mails",

    f_1: "Mais de 30 estilos de letras personalizadas",
    f_2: "Cópia rápida com um clique",
    f_3: "Compatível com Instagram, WhatsApp, Discord e jogos",
    f_4: "Gratuito, sem cadastro e sem instalação"
  }
}
</i18n>