<script setup lang="ts">
const { t, locale } = useI18n({ useScope: 'local' })
const localePath = useLocalePath()

usePageJsonLd({
  name: t('title'),
  description: t('meta'),
  type: 'tool',
  breadcrumb: [
    { name: 'Home', url: localePath('/') },
    { name: t('title') }
  ],
  features: [
    "Extract all valid email addresses from any text automatically",
    "Remove duplicate emails automatically",
    "Export with comma, line break, semicolon, or pipe separator",
    "Drag and drop file support"
  ]
})

useHead({
  title: t('title'),
  meta: [
    { name: "description", content: t('meta') }
  ]
})

// Tool State
const state = reactive({ 
  text: '', 
  output: null as string | null, 
  sep: ',', 
  loading: false 
})

const sepOptions = [
  { label: t('comma'), value: ',' },
  { label: t('line'), value: '||' },
  { label: t('colon'), value: ';' },
  { label: t('pipe'), value: '|' }
]

function extract() {
  if (state.text !== "") {
    state.loading = true;

    // Simulate small delay for UX
    setTimeout(() => {
      // Improved regex to avoid capturing trailing punctuation like dots or commas
      const regex = /[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9._-]+\.[a-zA-Z]{2,8}/gi;
      let emails: string[] | null = state.text.match(regex);

      if (emails && emails.length > 0) {
        // Cleaning potential duplicates and ensuring valid format
        state.output = [...new Set(emails)].join(state.sep !== "||" ? state.sep : "\n");
      } else {
        state.output = ''
      }

      state.loading = false;
    }, 100);
  }
}

defineI18nRoute({
  paths: {
    pt: "/extrator-de-emails",
    es: "/extractor-de-emails",
    fr: '/extracteur-emails',
    it: '/estrattore-di-email',
    id: '/ekstraktor-email'
  }
})
</script>

<template>
  <ToolPage
    :title="t('title')"
    :description="t('meta')"
    :show-ads="state.output !== null"
    :see-also-links="[
      { label: t('see1'), to: 'percentage-calculator' },
      { label: t('see2'), to: 'list-randomizer' },
      { label: t('see3'), to: 'dice-roller' },
      { label: t('see4'), to: 'simple-rule-of-three-calculator' }
    ]"
  >
    <template #info>
      <div class="space-y-4">
        <p>{{ t('desc') }}</p>
      </div>
    </template>

    <div class="space-y-6">
      <div class="form-control w-full">
        <label class="label">
          <span class="label-text font-bold text-base-content/80">{{ t('plc') }}</span>
        </label>
        <textarea 
          v-model="state.text" 
          rows="8" 
          class="textarea textarea-bordered textarea-lg w-full bg-base-200 focus:bg-base-200 transition-all rounded-2xl" 
          :placeholder="t('plc')"
          autofocus
        ></textarea>
      </div>

      <div class="flex flex-col sm:flex-row items-end gap-4">
        <ToolSelect 
          v-model="state.sep"
          :label="t('sep')"
          :options="sepOptions"
          class="sm:w-64"
        />

        <ButtonPrimary 
          @click="extract"
          icon="heroicons:envelope-20-solid"
          :is-loading="state.loading"
          :disabled="!state.text || state.loading"
        >
          {{ t('bt') }}
        </ButtonPrimary>
      </div>

      <!-- Drag and Drop -->
      <DragDropText @on-drop="txt => state.text = txt" />

      <!-- Result -->
      <Blockcopy 
        v-if="state.output" 
        :label="t('res')" 
        :content="state.output"
      >
        <pre class="whitespace-pre-wrap break-all font-mono text-sm">{{ state.output }}</pre>
      </Blockcopy>

      <!-- Empty State -->
      <Transition
        enter-active-class="transition duration-300 ease-out"
        enter-from-class="transform scale-95 opacity-0"
        enter-to-class="transform scale-100 opacity-100"
      >
        <div v-if="state.output === ''" class="alert alert-error shadow-lg border-none rounded-2xl bg-error/10 text-error">
          <Icon name="heroicons:information-circle-20-solid" class="w-6 h-6" />
          <span class="font-bold">{{ t('empty') }}</span>
        </div>
      </Transition>
    </div>
  </ToolPage>
</template>

<i18n lang="yaml">
{
    en: {
        title: "Email Extractor",
        meta: "Extract multiple email addresses from any large text automatically with our free online Email Extractor. Filter and clean your email lists instantly.",
        desc: "Need to copy dozens of contacts hidden within a large block of text or data? Our Email Extractor is the perfect tool for the job. Simply paste or drag a long text document into the box, choose your preferred list separator (comma, line break, semicolon, or vertical bar), and click 'Extract'. Our algorithm will automatically find all valid embedded emails, filter out any duplicates, and generate a clean, ready-to-use list for your marketing campaigns or mailing needs.",
        sep: "Separate by",
        empty: "No email found",
        bt: "Extract",
        plc: "Insert your text here or drag a file",
        comma: "Comma",
        line: "Line break",
        colon: "Semicolon",
        pipe: "Vertical bar (pipe)",
        res: "Result",
        see1: "Percentage Calculator",
        see2: "List Randomizer",
        see3: "Dice Roller",
        see4: "Rule of Three Calculator"
    },
    pt: {
        title: "Extrator de E-mails",
        meta: "Extraia múltiplos endereços de e-mail de qualquer texto grande automaticamente com nosso Extrator de E-mails online e remova duplicados.",
        desc: "Precisa extrair dezenas de contatos misturados no meio de uma grande massa de dados ou código? Nosso Extrator de E-mails é a ferramenta ideal. Basta inserir ou arrastar um texto longo e selecionar qual será o separador da sua lista (vírgula, quebra de linha, ponto e vírgula ou barra vertical). Ao clicar em 'Extrair', nosso algoritmo encontrará automaticamente todos os e-mails válidos embutidos, removendo quaisquer repetições e gerando uma lista limpa e pronta para uso em campanhas de marketing ou mailings.",
        sep: "Separar por",
        empty: "Nenhum e-mail encontrado",
        bt: "Extrair",
        plc: "Insira seu texto aqui ou arraste um arquivo",
        comma: "Vírgula",
        line: "Quebra de linha",
        colon: "Ponto e vírgula",
        pipe: "Barra vertical (pipe)",
        res: "Resultado",
        see1: "Calculadora de Porcentagem",
        see2: "Sorteador de Listas",
        see3: "Lançador de Dados",
        see4: "Calculadora de Regra de Três"
    },
    es: {
        title: "Extractor de Correos Electrónicos",
        meta: "Extrae direcciones de correo de cualquier texto o documento largo con nuestro Extractor de Emails online gratuito. Elimina duplicados al instante.",
        desc: "¿Necesitas recuperar decenas de contactos ocultos en medio de grandes bloques de texto o código? Nuestro Extractor de Correos Electrónicos es la herramienta ideal. Solo tienes que pegar o arrastrar tus datos y seleccionar tu separador preferido (coma, salto de línea, punto y coma o barra vertical). Al hacer clic en 'Extraer', nuestro sistema rastreará de forma automática todos los correos electrónicos válidos, eliminará las direcciones repetidas y generará una lista limpia para tus campañas.",
        sep: "Separar por",
        empty: "No se encontraron correos electrónicos",
        bt: "Extraer",
        plc: "Introduce tu texto aquí o arrastra un archivo",
        comma: "Coma",
        line: "Salto de línea",
        colon: "Punto y coma",
        pipe: "Barra vertical",
        res: "Resultado",
        see1: "Calculadora de Porcentaje",
        see2: "Aleatorizador de Listas",
        see3: "Lanzador de Dados",
        see4: "Calculadora de Regla de Tres"
    },
    fr: {
        title: "Extracteur d'Emails",
        meta: "Extrayez plusieurs adresses e-mail à partir d'un texte long automatiquement avec notre Extracteur gratuit en ligne.",
        desc: "Vous avez besoin de copier des dizaines de contacts noyés dans un vaste bloc de texte ou de code ? Notre Extracteur d'Emails est l'outil parfait. Il suffit de coller ou glisser vos données textuelles, puis de configurer un séparateur de liste (virgule, saut de ligne, point-virgule ou barre verticale). En cliquant sur 'Extraire', l'outil détectera automatiquement toutes les adresses e-mail valides, supprimera automatiquement les doublons et affichera une liste propre pour vos campagnes de diffusion.",
        sep: "Séparer par",
        empty: "Aucun email trouvé",
        bt: "Extraire",
        plc: "Insérez votre texte ici ou faites glisser un fichier",
        comma: "Virgule",
        line: "Saut de ligne",
        colon: "Point-virgule",
        pipe: "Barre verticale",
        res: "Résultat",
        see1: "Calculatrice de Pourcentage",
        see2: "Mélangeur de Listes",
        see3: "Lanceur de Dés",
        see4: "Calculatrice de Règle de Trois"
    },
    it: {
        title: "Estrattore di Email",
        meta: "Estrai automaticamente numerosi indirizzi email da un testo o codice di grandi dimensioni con il nostro Estrattore di Email gratuito.",
        desc: "Hai bisogno di salvare dozzine di contatti nascosti all'interno di un enorme blocco di testo o codice sorgente? Il nostro Estrattore di Email è lo strumento perfetto per te. Basta incollare le informazioni nella casella di testo e selezionare un separatore (virgola, interruzione di linea, punto e virgola o barra verticale). Cliccando su 'Estrai', l'algoritmo rileverà in automatico tutte le email valide al suo interno, eliminerà automaticamente i doppioni e creerà un elenco ordinato per i tuoi invii di massa.",
        sep: "Separare per",
        empty: "Nessuna email trovata",
        bt: "Estrai",
        plc: "Inserisci il tuo testo qui o trascina un file",
        comma: "Virgola",
        line: "Interruzione di linea",
        colon: "Punto e virgola",
        pipe: "Barra verticale",
        res: "Risultato",
        see1: "Calcolatrice di Percentuale",
        see2: "Randomizzatore di Liste",
        see3: "Lanciatore di Dadi",
        see4: "Calcolatrice di Regola di Tre"
    },
    id: {
        title: "Ekstraktor Email",
        meta: "Ekstrak beberapa alamat email dari teks besar apa pun secara otomatis dengan Ekstraktor Email online gratis kami. Filter dan bersihkan daftar email Anda secara instan.",
        desc: "Perlu menyalin puluhan kontak yang tersembunyi di dalam blok teks atau data yang besar? Ekstraktor Email kami adalah alat yang sempurna untuk tugas tersebut. Cukup tempel atau seret dokumen teks panjang ke dalam kotak, pilih pemisah daftar yang Anda inginkan (koma, jeda baris, titik koma, atau garis tegak), dan klik 'Ekstrak'. Algoritma kami akan secara otomatis menemukan semua email valid yang tertanam, menyaring duplikat, dan menghasilkan daftar yang bersih dan siap digunakan untuk kampanye pemasaran atau kebutuhan mailing Anda.",
        sep: "Pisahkan dengan",
        empty: "Email tidak ditemukan",
        bt: "Ekstrak",
        plc: "Masukkan teks Anda di sini atau seret file ke sini",
        comma: "Koma",
        line: "Jeda baris",
        colon: "Titik koma",
        pipe: "Garis tegak (pipe)",
        res: "Hasil",
        see1: "Kalkulator Persentase",
        see2: "Pengacak Daftar",
        see3: "Pelempar Dadu",
        see4: "Kalkulator Aturan Tiga"
    }
}
</i18n>
