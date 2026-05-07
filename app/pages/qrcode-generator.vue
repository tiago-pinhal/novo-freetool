<script setup lang="ts">
type QRCodeOptions = Record<string, unknown>

interface QRCodeStylingInstance {
  update: (options: QRCodeOptions) => void
  append: (element: HTMLElement) => void
  download: (options: { name: string; extension: 'png' }) => void
}

type QRCodeStylingConstructor = new (options: QRCodeOptions) => QRCodeStylingInstance

declare global {
  interface Window {
    QRCodeStyling?: QRCodeStylingConstructor
  }
}

const { t } = useI18n({ useScope: 'local' })


const state = reactive({
  type: 'URL',
  url: '',
  text: '',
  sms_phone: '',
  sms_message: '',
  email_address: '',
  email_subject: '',
  email_message: '',
  wifi_ssid: '',
  wifi_hidden: false,
  wifi_pass: '',
  wifi_cripto: 'WPA',
  card_first_name: '',
  card_last_name: '',
  card_cell: '',
  card_tel_work: '',
  card_email: '',
  card_street: '',
  card_city: '',
  card_state: '',
  card_zip: '',
  card_country: '',
  card_org: '',
  card_job: '',
  card_site: '',
  pix_key: '',
  pix_name: '',
  pix_city: '',
  pix_amount: '',
  pix_desc: '',
  width: 320,
  height: 320,
  lockProportions: true,
  margin: 8,
  logo: '',
  logo_hide: true,
  logo_margin: 5,
  dot_type: 'square',
  dot_color: '#111827',
  bg_color: '#ffffff',
  corners_type: 'square',
  corners_color: '#111827',
  corners_dot_type: 'square',
  corners_dot_color: '#111827'
})

const canvasRef = ref<HTMLElement | null>(null)
const fileInputRef = ref<HTMLInputElement | null>(null)
const qrLoaded = ref(false)
const downloadFeedback = ref('')
let qrCode: QRCodeStylingInstance | null = null
let logoObjectUrl = ''

const faqItems = computed(() => [
  { question: t('faq_1_q'), answer: t('faq_1_a') },
  { question: t('faq_3_q'), answer: t('faq_3_a') },
  { question: t('faq_4_q'), answer: t('faq_4_a') }
])

const hasCustomData = computed(() => {
  return Boolean(
    state.url ||
    state.text ||
    state.sms_phone ||
    state.sms_message ||
    state.email_address ||
    state.email_subject ||
    state.email_message ||
    state.wifi_ssid ||
    state.card_first_name ||
    state.card_last_name ||
    state.card_cell ||
    state.card_email
  )
})

usePageJsonLd({
  name: t('title'),
  description: t('meta'),
  features: [t('feature_1'), t('feature_2'), t('feature_3'), t('feature_4')],
  howToName: t('how_to_title'),
  howToSteps: [
    { name: t('how_1_title'), text: t('how_1_desc') },
    { name: t('how_2_title'), text: t('how_2_desc') },
    { name: t('how_3_title'), text: t('how_3_desc') }
  ],
  faq: faqItems.value
})

useHead({
  title: t('page_title'),
  meta: [
    { name: 'description', content: t('meta') },
    { property: 'og:title', content: t('og_title') },
    { property: 'og:description', content: t('meta') },
    { property: 'og:type', content: 'website' },
    { name: 'twitter:card', content: 'summary' },
    { name: 'twitter:title', content: t('og_title') },
    { name: 'twitter:description', content: t('meta') }
  ]
})

watch(() => state.width, (val) => {
  if (state.lockProportions) state.height = val
})

watch(() => state.height, (val) => {
  if (state.lockProportions) state.width = val
})

watch(state, () => {
  if (qrLoaded.value) generate()
}, { deep: true })

const previewContainerRef = ref<HTMLElement | null>(null)

onMounted(() => {
  // Auto-size based on container if it's the first load
  if (previewContainerRef.value) {
    const width = previewContainerRef.value.clientWidth - 24 // Subtract padding
    if (width > 120 && width < 1000) {
      state.width = Math.floor(width)
      state.height = Math.floor(width)
    }
  }
})

onBeforeUnmount(() => {
  if (logoObjectUrl) URL.revokeObjectURL(logoObjectUrl)
})

function removeAccents(str: string) {
  return str.normalize('NFD').replace(/[\u0300-\u036f]/g, '')
}

function generate() {
  if (!window.QRCodeStyling || !canvasRef.value) return

  const options: QRCodeOptions = {
    data: getData(),
    width: clampDimension(state.width),
    height: clampDimension(state.height),
    margin: Math.max(0, Number(state.margin) || 0),
    type: 'svg',
    image: state.logo || undefined,
    imageOptions: {
      hideBackgroundDots: state.logo_hide,
      margin: Math.max(0, Number(state.logo_margin) || 0),
      crossOrigin: 'anonymous'
    },
    dotsOptions: {
      type: state.dot_type,
      color: state.dot_color
    },
    cornersSquareOptions: {
      type: state.corners_type,
      color: state.corners_color
    },
    cornersDotOptions: {
      type: state.corners_dot_type,
      color: state.corners_dot_color
    },
    backgroundOptions: {
      color: state.bg_color
    }
  }

  if (qrCode) {
    qrCode.update(options)
  } else {
    qrCode = new window.QRCodeStyling(options)
    qrCode.append(canvasRef.value)
  }
}

function clampDimension(value: number) {
  return Math.min(1000, Math.max(120, Number(value) || 320))
}

function getData() {
  switch (state.type) {
    case 'URL':
      return state.url.trim() || 'https://www.freetool.dev'
    case 'VCARD':
      return getVCard()
    case 'EMAIL':
      return `MATMSG:TO:${escapeQrValue(state.email_address)};SUB:${escapeQrValue(state.email_subject)};BODY:${escapeQrValue(state.email_message)};;`
    case 'WIFI':
      return `WIFI:S:${escapeQrValue(state.wifi_ssid)};P:${escapeQrValue(state.wifi_pass)};T:${state.wifi_cripto};H:${state.wifi_hidden};`
    case 'SMS':
       return `SMSTO:${state.sms_phone}:${state.sms_message}`
    case 'PIX':
       return generatePixPayload()
     default:
       return state.text.trim() || t('text_placeholder')
   }
 }
 
 function generatePixPayload() {
   const pixKey = state.pix_key.trim()
   if (!pixKey) return ''
 
   const name = removeAccents(state.pix_name.trim() || 'RECEBEDOR').toUpperCase().substring(0, 25)
   const city = removeAccents(state.pix_city.trim() || 'CIDADE').toUpperCase().substring(0, 15)
   const amount = state.pix_amount ? Number(state.pix_amount).toFixed(2) : ''
   const desc = removeAccents(state.pix_desc.trim()).substring(0, 20)
 
   // Merchant Account Information
   let merchantInfo = '0014br.gov.bcb.pix'
   merchantInfo += `01${pixKey.length.toString().padStart(2, '0')}${pixKey}`
   if (desc) {
     merchantInfo += `02${desc.length.toString().padStart(2, '0')}${desc}`
   }
 
   let payload = '000201' // Payload Format Indicator
   payload += `26${merchantInfo.length.toString().padStart(2, '0')}${merchantInfo}`
   payload += '52040000' // Merchant Category Code
   payload += '5303986'  // Transaction Currency (BRL)
   
   if (amount && amount !== '0.00') {
     payload += `54${amount.length.toString().padStart(2, '0')}${amount}`
   }
 
   payload += '5802BR'   // Country Code
   payload += `59${name.length.toString().padStart(2, '0')}${name}`
   payload += `60${city.length.toString().padStart(2, '0')}${city}`
   payload += '62070503***' // Additional Data Field (Reference Label)
   payload += '6304'      // CRC16 Indicator
 
   return payload + calculateCRC16(payload)
 }
 
 function calculateCRC16(data: string) {
   let crc = 0xFFFF
   const polynomial = 0x1021
 
   for (let i = 0; i < data.length; i++) {
     crc ^= data.charCodeAt(i) << 8
     for (let j = 0; j < 8; j++) {
       if (crc & 0x8000) {
         crc = (crc << 1) ^ polynomial
       } else {
         crc = crc << 1
       }
     }
   }
 
   return (crc & 0xFFFF).toString(16).toUpperCase().padStart(4, '0')
 }

function getVCard() {
  let content = 'BEGIN:VCARD\nVERSION:4.0'
  const fullName = [state.card_first_name, state.card_last_name].filter(Boolean).join(' ')

  if (fullName) content += `\nFN:${escapeVCard(fullName)}`
  if (state.card_last_name || state.card_first_name) {
    content += `\nN:${escapeVCard(state.card_last_name)};${escapeVCard(state.card_first_name)};;;`
  }
  if (state.card_street || state.card_city || state.card_state || state.card_zip || state.card_country) {
    content += `\nADR:;;${escapeVCard(state.card_street)};${escapeVCard(state.card_city)};${escapeVCard(state.card_state)};${escapeVCard(state.card_zip)};${escapeVCard(state.card_country)}`
  }
  if (state.card_cell) content += `\nTEL;TYPE=cell:tel:${escapeVCard(state.card_cell)}`
  if (state.card_tel_work) content += `\nTEL;TYPE=work:tel:${escapeVCard(state.card_tel_work)}`
  if (state.card_email) content += `\nEMAIL:${escapeVCard(state.card_email)}`
  if (state.card_org) content += `\nORG:${escapeVCard(state.card_org)}`
  if (state.card_job) content += `\nTITLE:${escapeVCard(state.card_job)}`
  if (state.card_site) content += `\nURL:${escapeVCard(state.card_site)}`

  return `${content}\nEND:VCARD`
}

function escapeVCard(value: string) {
  return value.replace(/\\/g, '\\\\').replace(/\n/g, '\\n').replace(/,/g, '\\,').replace(/;/g, '\\;')
}

function escapeQrValue(value: string) {
  return value.replace(/\\/g, '\\\\').replace(/;/g, '\\;').replace(/:/g, '\\:')
}

function chooseLogo() {
  fileInputRef.value?.click()
}

function onLogoChange(event: Event) {
  const input = event.target as HTMLInputElement
  const file = input.files?.[0]
  if (!file) return

  if (logoObjectUrl) URL.revokeObjectURL(logoObjectUrl)
  logoObjectUrl = URL.createObjectURL(file)
  state.logo = logoObjectUrl
}

function removeLogo() {
  if (logoObjectUrl) URL.revokeObjectURL(logoObjectUrl)
  logoObjectUrl = ''
  state.logo = ''
  if (fileInputRef.value) fileInputRef.value.value = ''
}

function download() {
  if (!qrCode) return
  qrCode.download({ name: 'qrcode', extension: 'png' })
  downloadFeedback.value = t('download_success')
  window.setTimeout(() => {
    downloadFeedback.value = ''
  }, 2000)
}

defineI18nRoute({
  paths: {
    pt: '/gerador-de-qr-code',
    es: '/generador-de-qr-code',
    fr: '/generateur-de-qr-code',
    it: '/generatore-di-qr-code',
    id: '/pembuat-kode-qr'
  }
})

const { onLoaded } = useScript('https://cdn.jsdelivr.net/npm/@charles-goode/qr-code-styling@1.6.2/lib/qr-code-styling.js', {
  trigger: 'client'
})

onLoaded(() => {
  qrLoaded.value = true
  generate()
})
</script>

<template>
  <ToolPage
    :title="t('title')"
    :description="t('meta')"
    :show-ads="hasCustomData"
    :see-also-links="[
      { label: t('see1'), to: 'emoji-picker' },
      { label: t('see2'), to: 'barcode-generator' },
      { label: t('see3'), to: 'random-colors' },
      { label: t('see4'), to: 'dominant-colors-of-the-image' }
    ]"
  >
    <div class="grid gap-6 lg:grid-cols-[minmax(0,1fr)_360px]">
      <div class="space-y-4">
        <section class="rounded-2xl border border-base-content/10 bg-base-200/35 p-4">
          <h2 class="mb-4 text-lg font-bold text-base-content">{{ t('data') }}</h2>

          <div class="grid gap-4">
            <label class="form-control flex flex-col w-full">
              <span class="label-text mb-2 font-semibold">{{ t('data_opt') }}</span>
              <select id="qr-type" v-model="state.type" class="select select-bordered bg-base-100">
                <option value="URL">URL / Link</option>
                <option value="VCARD">vCard</option>
                <option value="EMAIL">Email</option>
                <option value="WIFI">Wi-Fi</option>
                <option value="SMS">SMS</option>
                <option value="PIX">PIX (Brasil)</option>
                <option value="TEXT">{{ t('text') }}</option>
              </select>
            </label>

            <label v-if="state.type === 'URL'" class="form-control flex flex-col w-full">
              <span class="label-text mb-2 font-semibold">URL</span>
              <input id="qr-url" v-model="state.url" type="url" class="input input-bordered bg-base-100" placeholder="https://www.freetool.dev" inputmode="url" />
            </label>

            <template v-if="state.type === 'VCARD'">
              <div class="grid gap-3 sm:grid-cols-2">
                <label class="form-control flex flex-col w-full">
                  <span class="label-text mb-2 font-semibold">{{ t('first_name') }}</span>
                  <input id="first-name" v-model="state.card_first_name" type="text" class="input input-bordered bg-base-100" autocomplete="given-name" />
                </label>
                <label class="form-control flex flex-col w-full">
                  <span class="label-text mb-2 font-semibold">{{ t('last_name') }}</span>
                  <input id="last-name" v-model="state.card_last_name" type="text" class="input input-bordered bg-base-100" autocomplete="family-name" />
                </label>
              </div>

              <div class="grid gap-3 sm:grid-cols-2">
                <label class="form-control flex flex-col w-full">
                  <span class="label-text mb-2 font-semibold">{{ t('cell') }}</span>
                  <input id="cell" v-model="state.card_cell" type="tel" class="input input-bordered bg-base-100" autocomplete="tel" />
                </label>
                <label class="form-control flex flex-col w-full">
                  <span class="label-text mb-2 font-semibold">{{ t('tel_work') }}</span>
                  <input id="tel-work" v-model="state.card_tel_work" type="tel" class="input input-bordered bg-base-100" />
                </label>
              </div>

              <label class="form-control flex flex-col w-full">
                <span class="label-text mb-2 font-semibold">Email</span>
                <input id="card-email" v-model="state.card_email" type="email" class="input input-bordered bg-base-100" autocomplete="email" />
              </label>

              <div class="grid gap-3 sm:grid-cols-2">
                <label class="form-control flex flex-col w-full">
                  <span class="label-text mb-2 font-semibold">{{ t('org') }}</span>
                  <input id="org" v-model="state.card_org" type="text" class="input input-bordered bg-base-100" autocomplete="organization" />
                </label>
                <label class="form-control flex flex-col w-full">
                  <span class="label-text mb-2 font-semibold">{{ t('job') }}</span>
                  <input id="job" v-model="state.card_job" type="text" class="input input-bordered bg-base-100" />
                </label>
              </div>

              <details class="collapse collapse-arrow rounded-2xl border border-base-content/10 bg-base-100">
                <summary class="collapse-title font-semibold">{{ t('address') }}</summary>
                <div class="collapse-content grid gap-3">
                  <label class="form-control flex flex-col w-full">
                    <span class="label-text mb-2 font-semibold">{{ t('street') }}</span>
                    <input v-model="state.card_street" type="text" class="input input-bordered bg-base-100" :placeholder="t('street')" autocomplete="street-address" />
                  </label>
                  <div class="grid gap-3 sm:grid-cols-2">
                    <label class="form-control flex flex-col w-full">
                      <span class="label-text mb-2 font-semibold">{{ t('city') }}</span>
                      <input v-model="state.card_city" type="text" class="input input-bordered bg-base-100" :placeholder="t('city')" autocomplete="address-level2" />
                    </label>
                    <label class="form-control flex flex-col w-full">
                      <span class="label-text mb-2 font-semibold">{{ t('state') }}</span>
                      <input v-model="state.card_state" type="text" class="input input-bordered bg-base-100" :placeholder="t('state')" autocomplete="address-level1" />
                    </label>
                  </div>
                  <div class="grid gap-3 sm:grid-cols-2">
                    <label class="form-control flex flex-col w-full">
                      <span class="label-text mb-2 font-semibold">{{ t('zip') }}</span>
                      <input v-model="state.card_zip" type="text" class="input input-bordered bg-base-100" :placeholder="t('zip')" autocomplete="postal-code" />
                    </label>
                    <label class="form-control flex flex-col w-full">
                      <span class="label-text mb-2 font-semibold">{{ t('country') }}</span>
                      <input v-model="state.card_country" type="text" class="input input-bordered bg-base-100" :placeholder="t('country')" autocomplete="country-name" />
                    </label>
                  </div>
                  <label class="form-control flex flex-col w-full">
                    <span class="label-text mb-2 font-semibold">Site</span>
                    <input v-model="state.card_site" type="url" class="input input-bordered bg-base-100" placeholder="Site" inputmode="url" />
                  </label>
                </div>
              </details>
            </template>

            <template v-if="state.type === 'EMAIL'">
              <label class="form-control flex flex-col w-full">
                <span class="label-text mb-2 font-semibold">Email</span>
                <input v-model="state.email_address" type="email" class="input input-bordered bg-base-100" placeholder="email@exemplo.com" autocomplete="email" />
              </label>
              <label class="form-control flex flex-col w-full">
                <span class="label-text mb-2 font-semibold">{{ t('subject') }}</span>
                <input v-model="state.email_subject" type="text" class="input input-bordered bg-base-100" :placeholder="t('subject')" />
              </label>
              <label class="form-control flex flex-col w-full">
                <span class="label-text mb-2 font-semibold">{{ t('msg') }}</span>
                <textarea v-model="state.email_message" class="textarea textarea-bordered min-h-28 bg-base-100" :placeholder="t('msg')" maxlength="4000" />
              </label>
            </template>

            <template v-if="state.type === 'WIFI'">
              <label class="form-control flex flex-col w-full">
                <span class="label-text mb-2 font-semibold">{{ t('ssid') }}</span>
                <input v-model="state.wifi_ssid" type="text" class="input input-bordered bg-base-100" :placeholder="t('ssid')" autocomplete="off" />
              </label>
              <div class="grid gap-3 sm:grid-cols-2">
                <label class="form-control flex flex-col w-full">
                  <span class="label-text mb-2 font-semibold">{{ t('pass') }}</span>
                  <input v-model="state.wifi_pass" type="password" class="input input-bordered bg-base-100" :placeholder="t('pass')" autocomplete="new-password" />
                </label>
                <label class="form-control flex flex-col w-full">
                  <span class="label-text mb-2 font-semibold">{{ t('cripto') }}</span>
                  <select v-model="state.wifi_cripto" class="select select-bordered bg-base-100" :aria-label="t('cripto')">
                    <option value="WPA">WPA/WPA2/WPA3</option>
                    <option value="WEP">WEP</option>
                    <option value="nopass">{{ t('none') }}</option>
                  </select>
                </label>
              </div>
              <label class="label cursor-pointer justify-start gap-3">
                <input v-model="state.wifi_hidden" type="checkbox" class="toggle toggle-primary" />
                <span class="label-text">{{ t('hidden') }}</span>
              </label>
            </template>

            <template v-if="state.type === 'SMS'">
              <label class="form-control flex flex-col w-full">
                <span class="label-text mb-2 font-semibold">{{ t('phone') }}</span>
                <input v-model="state.sms_phone" type="tel" class="input input-bordered bg-base-100" :placeholder="t('phone')" inputmode="tel" />
              </label>
              <label class="form-control flex flex-col w-full">
                <span class="label-text mb-2 font-semibold">{{ t('msg') }}</span>
                <textarea v-model="state.sms_message" class="textarea textarea-bordered min-h-24 bg-base-100" :placeholder="t('msg')" maxlength="150" />
              </label>
            </template>

            <template v-if="state.type === 'PIX'">
              <label class="form-control flex flex-col w-full">
                <span class="label-text mb-2 font-semibold">{{ t('pix_key') }}</span>
                <input v-model="state.pix_key" type="text" class="input input-bordered bg-base-100" placeholder="CPF, Email, Telefone ou Chave Aleatória" />
              </label>
              <div class="grid gap-3 sm:grid-cols-2">
                <label class="form-control flex flex-col w-full">
                  <span class="label-text mb-2 font-semibold">{{ t('pix_name') }}</span>
                  <input v-model="state.pix_name" type="text" class="input input-bordered bg-base-100" placeholder="Nome do Recebedor" maxlength="25" />
                </label>
                <label class="form-control flex flex-col w-full">
                  <span class="label-text mb-2 font-semibold">{{ t('pix_city') }}</span>
                  <input v-model="state.pix_city" type="text" class="input input-bordered bg-base-100" placeholder="Cidade" maxlength="15" />
                </label>
              </div>
              <div class="grid gap-3 sm:grid-cols-2">
                <label class="form-control flex flex-col w-full">
                  <span class="label-text mb-2 font-semibold">{{ t('pix_amount') }} (Opcional)</span>
                  <input v-model="state.pix_amount" type="number" step="0.01" min="0" class="input input-bordered bg-base-100" placeholder="0,00" />
                </label>
                <label class="form-control flex flex-col w-full">
                  <span class="label-text mb-2 font-semibold">{{ t('msg') }} (Opcional)</span>
                  <input v-model="state.pix_desc" type="text" class="input input-bordered bg-base-100" placeholder="Descrição" maxlength="20" />
                </label>
              </div>
              <p class="text-xs text-base-content/60 px-1 mt-1">
                Nota: O QR Code PIX gerado é estático. Verifique os dados antes de compartilhar.
              </p>
            </template>

            <label v-if="state.type === 'TEXT'" class="form-control flex flex-col w-full">
              <span class="label-text mb-2 font-semibold">{{ t('text') }}</span>
              <textarea
                v-model="state.text"
                class="textarea textarea-bordered min-h-28 bg-base-100"
                :placeholder="t('text_placeholder')"
                maxlength="4000"
              />
            </label>
          </div>
        </section>

        <section class="rounded-2xl border border-base-content/10 bg-base-200/35 p-4">
          <h2 class="mb-4 text-lg font-bold text-base-content">{{ t('customization') }}</h2>

          <div class="grid gap-5">
            <div class="flex items-end gap-3">
              <label class="form-control flex-1">
                <span class="label-text mb-2 font-semibold">{{ t('width') }}</span>
                <input v-model.number="state.width" type="number" min="120" max="1000" class="input input-bordered bg-base-100" />
              </label>
              
              <div class="mb-2">
                <button 
                  type="button" 
                  class="btn btn-ghost btn-sm btn-circle" 
                  :class="{ 'text-primary': state.lockProportions }"
                  @click="state.lockProportions = !state.lockProportions"
                  :title="state.lockProportions ? 'Desvincular proporções' : 'Vincular proporções'"
                >
                  <Icon :name="state.lockProportions ? 'heroicons:link-20-solid' : 'heroicons:link-slash-20-solid'" class="h-5 w-5" />
                </button>
              </div>

              <label class="form-control flex-1">
                <span class="label-text mb-2 font-semibold">{{ t('height') }}</span>
                <input v-model.number="state.height" type="number" min="120" max="1000" class="input input-bordered bg-base-100" />
              </label>
              <label class="form-control w-20">
                <span class="label-text mb-2 font-semibold">{{ t('margin') }}</span>
                <input v-model.number="state.margin" type="number" min="0" max="80" class="input input-bordered bg-base-100" />
              </label>
            </div>

            <div class="grid gap-3 sm:grid-cols-2">
              <label class="form-control flex flex-col w-full">
                <span class="label-text mb-2 font-semibold">{{ t('dot_style') }}</span>
                <select v-model="state.dot_type" class="select select-bordered bg-base-100">
                  <option value="square">{{ t('square') }}</option>
                  <option value="dots">{{ t('dots') }}</option>
                  <option value="rounded">{{ t('rounded') }}</option>
                  <option value="extra-rounded">Extra {{ t('rounded') }}</option>
                  <option value="classy">{{ t('classy') }}</option>
                  <option value="classy-rounded">{{ t('classy') }} {{ t('rounded') }}</option>
                </select>
              </label>
              <label class="form-control flex flex-col w-full">
                <span class="label-text mb-2 font-semibold">{{ t('corner_style') }}</span>
                <select v-model="state.corners_type" class="select select-bordered bg-base-100">
                  <option value="square">{{ t('square') }}</option>
                  <option value="dots">{{ t('dots') }}</option>
                  <option value="extra-rounded">Extra {{ t('rounded') }}</option>
                </select>
              </label>
            </div>

            <h3 class="text-sm font-bold opacity-80 uppercase tracking-wide">{{ t('colors_label') }}</h3>

            <div class="grid gap-3 grid-cols-2">
              <label class="qr-color-control">
                <span>{{ t('dot_color') }}</span>
                <input v-model="state.dot_color" type="color" />
              </label>
              <label class="qr-color-control">
                <span>{{ t('corners_color') }}</span>
                <input v-model="state.corners_color" type="color" />
              </label>
              <label class="qr-color-control">
                <span>{{ t('corners_dot_color') }}</span>
                <input v-model="state.corners_dot_color" type="color" />
              </label>
              <label class="qr-color-control">
                <span>{{ t('bck_clr') }}</span>
                <input v-model="state.bg_color" type="color" />
              </label>
            </div>

            <div class="rounded-2xl border border-base-content/10 bg-base-100 p-4">
              <div class="flex flex-wrap items-center justify-between gap-3">
                <div>
                  <h3 class="font-bold">{{ t('logo') }}</h3>
                  <p class="text-sm text-base-content/70">{{ t('logo_help') }}</p>
                </div>
                <div class="flex flex-wrap gap-2">
                  <button type="button" class="btn btn-primary btn-sm" @click="chooseLogo">
                    <Icon name="heroicons:arrow-up-tray-20-solid" class="h-4 w-4" aria-hidden="true" />
                    {{ t('choose_logo') }}
                  </button>
                  <button v-if="state.logo" type="button" class="btn btn-ghost btn-sm" @click="removeLogo">
                    {{ t('remove') }}
                  </button>
                </div>
              </div>

              <input ref="fileInputRef" type="file" accept="image/*" class="hidden" @change="onLogoChange" />

              <div v-if="state.logo" class="mt-4 grid gap-3 sm:grid-cols-[1fr_160px]">
                <label class="label cursor-pointer justify-start gap-3 rounded-xl bg-base-200 px-3">
                  <input v-model="state.logo_hide" type="checkbox" class="toggle toggle-primary" />
                  <span class="label-text">{{ t('hide') }}</span>
                </label>
                <label class="form-control flex flex-col w-full">
                  <span class="label-text mb-2 font-semibold">{{ t('logo_margin') }}</span>
                  <input v-model.number="state.logo_margin" type="number" min="0" max="40" class="input input-bordered bg-base-100" />
                </label>
              </div>
            </div>
          </div>
        </section>
      </div>

      <aside class="lg:sticky lg:top-6 lg:self-start">
        <div class="rounded-2xl border border-primary/20 bg-base-100 p-4 shadow-sm">
          <div class="mb-3 flex items-center justify-between gap-3">
            <h2 class="text-lg font-bold">{{ t('preview') }}</h2>
            <span class="badge badge-outline">{{ state.width }} x {{ state.height }}</span>
          </div>

          <div ref="previewContainerRef" class="flex flex-col min-h-[340px] items-center justify-center rounded-2xl bg-base-200/50 p-3 overflow-hidden">
            <div id="qr-code" ref="canvasRef" class="qr-preview overflow-hidden rounded-xl bg-white shadow-inner flex items-center justify-center min-h-[300px]" />
            
            <div v-if="state.width > 400 || state.height > 400" class="mt-2 text-center">
              <span class="badge badge-ghost badge-sm opacity-60">Visualização em escala ({{ state.width }}x{{ state.height }}px)</span>
            </div>
          </div>

          <button type="button" class="btn btn-primary mt-4 w-full" :disabled="!qrLoaded" :aria-label="t('download_aria')" @click="download">
            <Icon name="heroicons:arrow-down-tray-20-solid" class="h-5 w-5" aria-hidden="true" />
            Download PNG
          </button>

          <p class="mt-3 text-sm text-base-content/70">{{ t('scan_tip') }}</p>
          <div role="status" aria-live="polite" class="sr-only">{{ downloadFeedback }}</div>
        </div>
      </aside>
    </div>

    <template #info>
      <div class="space-y-8">
        <p>{{ t('intro') }}</p>

        <FeatureSection
          :title="t('features_title')"
          :items="[t('feature_1'), t('feature_2'), t('feature_3'), t('feature_4')]"
        />

        <UseCaseSection
          :title="t('use_cases_title')"
          :description="t('use_cases_desc')"
          :items="[
            { title: t('uc_1_title'), description: t('uc_1_desc') },
            { title: t('uc_2_title'), description: t('uc_2_desc') },
            { title: t('uc_3_title'), description: t('uc_3_desc') },
            { title: t('uc_4_title'), description: t('uc_4_desc') }
          ]"
        />

        <HowToSection
          :title="t('how_to_title')"
          :items="[
            { title: t('how_1_title'), description: t('how_1_desc') },
            { title: t('how_2_title'), description: t('how_2_desc') },
            { title: t('how_3_title'), description: t('how_3_desc') }
          ]"
        />

        <section>
          <h2 class="mb-3 text-2xl font-bold text-base-content">{{ t('types_title') }}</h2>
          <p class="mb-4 text-base-content/80 leading-relaxed">{{ t('types_desc') }}</p>
          <div class="grid gap-3 sm:grid-cols-2">
            <p v-for="key in ['type_url', 'type_vcard', 'type_pix', 'type_email', 'type_wifi', 'type_sms', 'type_text']" :key="key" class="rounded-2xl border border-base-content/10 bg-base-200/40 p-4">
              {{ t(key) }}
            </p>
          </div>
        </section>

        <FaqSection :title="t('faq_title')" :items="faqItems" />
      </div>
    </template>
  </ToolPage>
</template>

<style scoped>
.qr-preview {
  width: 100%;
  max-width: 100%;
}

.qr-preview :deep(svg), 
.qr-preview :deep(canvas) {
  max-width: 100% !important;
  height: auto !important;
  display: block;
}

.qr-preview :deep(svg),
.qr-preview :deep(canvas) {
  max-width: 100% !important;
  max-height: 100% !important;
  height: auto !important;
  width: auto !important;
  display: block;
}

/* Ensure the library's wrapper div also behaves */
.qr-preview :deep(> div) {
  display: flex !important;
  align-items: center;
  justify-content: center;
  max-width: 100%;
  max-height: 100%;
}

.qr-color-control {
  display: flex;
  min-height: 3.25rem;
  align-items: center;
  justify-content: space-between;
  gap: 0.75rem;
  border: 1px solid color-mix(in oklab, currentColor 12%, transparent);
  border-radius: 1rem;
  background: color-mix(in oklab, currentColor 4%, transparent);
  padding: 0.75rem;
  font-size: 0.875rem;
  font-weight: 600;
}

.qr-color-control input {
  width: 2.25rem;
  height: 2.25rem;
  flex: none;
  cursor: pointer;
  border: 0;
  border-radius: 999px;
  background: transparent;
  padding: 0;
}
</style>

<i18n lang="yaml">
{
  en: {
    title: "QR Code Generator",
    page_title: "QR Code Generator with Logo - Create Free Custom QR Code",
    meta: "Create free custom QR codes for URL, vCard, Wi-Fi, PIX, Email, SMS and text. Adjust colors, shapes, size, logo and download your high-resolution image.",
    og_title: "Free Online QR Code Generator with Logo",
    intro: "Create custom QR codes simply and for free, with real-time preview and high-resolution image download. Generate the main types of QR codes with full control over colors, styles, and logos.",
    data: "QR Code Data",
    data_opt: "Data type",
    text: "Text",
    text_placeholder: "Type the text that will be saved in the QR Code",
    first_name: "First Name",
    last_name: "Last Name",
    address: "Address",
    street: "Street",
    city: "City",
    state: "State",
    zip: "Zip Code",
    country: "Country",
    phone: "Phone",
    cell: "Cell",
    tel_work: "Work Phone",
    org: "Organization",
    job: "Job Title",
    subject: "Subject",
    msg: "Message",
    ssid: "Network Name",
    cripto: "Encryption",
    none: "None",
    hidden: "Hidden Wi-Fi network",
    pass: "Password",
    pix_key: "PIX Key",
    pix_name: "Receiver Name",
    pix_city: "City",
    pix_amount: "Amount",
    colors_label: "Colors",
    customization: "Customization",
    width: "Width",
    height: "Height",
    margin: "Margin",
    logo: "Logo",
    logo_help: "Add an image to the center of the QR Code and test reading before printing.",
    choose_logo: "Choose logo",
    logo_margin: "Logo margin",
    hide: "Hide dots behind logo",
    remove: "Remove",
    dot_style: "Dot style",
    corner_style: "Corner style",
    square: "Square",
    dots: "Dots",
    rounded: "Rounded",
    classy: "Classy",
    dot_color: "Dots",
    corners_color: "Corners",
    corners_dot_color: "Corner center",
    bck_clr: "Background",
    preview: "Preview",
    scan_tip: "Tip: use good contrast between dots and background to keep the QR Code easy to scan.",
    download_aria: "Download QR Code as PNG image",
    download_success: "QR Code downloaded successfully.",
    features_title: "QR Code generator features",
    feature_1: "QR Code creation for URL, vCard contacts, Wi-Fi, PIX, Email, SMS and text",
    feature_2: "Customization of colors, dot shapes, corners, size and margin",
    feature_3: "Central logo upload with margin control and hidden background",
    feature_4: "Real-time preview and PNG download",
    use_cases_title: "Where to use custom QR Codes",
    use_cases_desc: "Custom QR codes help connect physical materials to digital experiences without requiring the person to manually type links or data.",
    uc_1_title: "Marketing and printed materials",
    uc_1_desc: "Include QR codes on cards, folders, packaging, banners and posters to lead visitors to pages, catalogs, coupons and campaigns.",
    uc_2_title: "Digital business card",
    uc_2_desc: "Use the vCard template to share name, phone, email, company, job title, address and website in a single scan.",
    uc_3_title: "Quick Wi-Fi access",
    uc_3_desc: "Create QR codes for Wi-Fi networks and allow customers, guests or teams to connect without typing the password.",
    uc_4_title: "Service and communication",
    uc_4_desc: "Generate QR codes for pre-filled SMS or email and reduce steps in forms, support, reservations and quote requests.",
    how_to_title: "How to create a QR Code",
    how_1_title: "Choose content type",
    how_1_desc: "Select a content type and fill in only the necessary fields for that format.",
    how_2_title: "Customize appearance",
    how_2_desc: "Adjust size, margin, colors, dot shape and corners. If you want, add your brand logo to the center.",
    how_3_title: "Test and download in PNG",
    how_3_desc: "Check the preview, scan with your phone to validate and download the PNG file ready to use digitally or in print.",
    types_title: "Available QR Code types",
    types_desc: "Each type generates a structure compatible with common QR code readers on modern smartphones.",
    type_url: "URL: directs the person to a website, landing page, form, menu, social profile or online file.",
    type_vcard: "vCard: saves contact data directly to the phone's agenda.",
    type_email: "Email: opens the email app with recipient, subject and message already filled in.",
    type_wifi: "Wi-Fi: facilitates connection to a network with SSID, password and encryption type.",
    type_sms: "SMS: opens a text message with phone and content defined.",
    type_pix: "PIX: generates a static QR Code for receiving payments and transfers.",
    type_text: "Text: stores simple information that appears directly upon scanning.",
    faq_title: "Frequently Asked Questions",
    faq_1_q: "Can I add my logo to the QR Code?",
    faq_1_a: "Yes. Use the logo option to upload an image and position it in the center. To keep reading reliable, prefer simple logos and test the QR code before printing.",
    faq_3_q: "Does the QR Code still work with different colors and shapes?",
    faq_3_a: "Yes, as long as there is enough contrast between the dots and the background. Avoid very light colors on dots and always test reading on your phone.",
    faq_4_q: "What format is the download in?",
    faq_4_a: "Files are generated in high-quality PNG format, ideal for digital use and printing.",
    see1: "Emoji Picker",
    see2: "Barcode Generator",
    see3: "Random Colors",
    see4: "Dominant Colors of the Image",
  },
  pt: {
    title: "Gerador de QR Code",
    page_title: "Gerador de QR Code com Logo - Criar QR Code Personalizado Grátis",
    meta: "Crie QR Codes personalizados grátis para URL, vCard, Wi-Fi, PIX, Email, SMS e texto. Ajuste cores, formatos, tamanho, logo e baixe sua imagem em alta resolução.",
    og_title: "Gerador de QR Code com Logo Online e Grátis",
    intro: "Crie QR Codes personalizados de forma simples e gratuita, com pré-visualização em tempo real e download de imagem em alta resolução. Gere os principais tipos de QR Codes com total controle sobre cores, estilos e logotipos.",

    data: "Dados do QR Code",
    data_opt: "Tipo de dado",
    text: "Texto",
    text_placeholder: "Digite o texto que será salvo no QR Code",
    first_name: "Nome",
    last_name: "Sobrenome",
    address: "Endereço",
    street: "Rua",
    city: "Cidade",
    state: "Estado",
    zip: "Código Postal (CEP)",
    country: "País",
    phone: "Telefone",
    cell: "Celular",
    tel_work: "Telefone do trabalho",
    org: "Empresa",
    job: "Cargo",
    subject: "Assunto",
    msg: "Mensagem",
    ssid: "Nome da rede",
    cripto: "Criptografia",
    none: "Nenhuma",
    hidden: "Rede Wi-Fi oculta",
    pass: "Senha",
    pix_key: "Chave PIX",
    pix_name: "Nome do Recebedor",
    pix_city: "Cidade",
    pix_amount: "Valor",

    colors_label: "Cores",
    customization: "Personalização",
    width: "Largura",
    height: "Altura",
    margin: "Margem",
    logo: "Logotipo",
    logo_help: "Adicione uma imagem ao centro do QR Code e teste a leitura antes de imprimir.",
    choose_logo: "Escolher logo",
    logo_margin: "Margem do logo",
    hide: "Ocultar pontos atrás do logo",
    remove: "Remover",
    dot_style: "Formato dos pontos",
    corner_style: "Formato dos cantos",
    square: "Quadrado",
    dots: "Pontos",
    rounded: "Arredondado",
    classy: "Elegante",
    dot_color: "Pontos",
    corners_color: "Cantos",
    corners_dot_color: "Centro dos cantos",
    bck_clr: "Fundo",
    preview: "Pré-visualização",
    scan_tip: "Dica: use bom contraste entre pontos e fundo para manter o QR Code fácil de escanear.",
    download_aria: "Baixar QR Code como imagem PNG",
    download_success: "QR Code baixado com sucesso.",

    features_title: "Recursos do gerador de QR Code",
    feature_1: "Criação de QR Code para URL, contatos vCard, Wi-Fi, PIX, Email, SMS e texto",
    feature_2: "Personalização de cores, formatos dos pontos, cantos, tamanho e margem",
    feature_3: "Upload de logo central com controle de margem e fundo oculto",
    feature_4: "Prévia em tempo real e download em PNG",

    use_cases_title: "Onde usar QR Codes personalizados",
    use_cases_desc: "QR Codes personalizados ajudam a conectar materiais físicos a experiências digitais sem exigir que a pessoa digite links ou dados manualmente.",
    uc_1_title: "Marketing e materiais impressos",
    uc_1_desc: "Inclua QR Codes em cartões, folders, embalagens, banners e cartazes para levar visitantes a páginas, catálogos, cupons e campanhas.",
    uc_2_title: "Cartão de visita digital",
    uc_2_desc: "Use o modelo vCard para compartilhar nome, telefone, email, empresa, cargo, endereço e site em um único escaneamento.",
    uc_3_title: "Acesso rápido ao Wi-Fi",
    uc_3_desc: "Crie QR Codes para redes Wi-Fi e permita que clientes, convidados ou equipes se conectem sem digitar a senha.",
    uc_4_title: "Atendimento e comunicação",
    uc_4_desc: "Gere QR Codes para SMS ou email pré-preenchido e reduza etapas em formulários, suporte, reservas e pedidos de orçamento.",

    how_to_title: "Como criar um QR Code",
    how_1_title: "Escolha o tipo de conteúdo",
    how_1_desc: "Selecione um tipo de conteúdo e preencha apenas os campos necessários para aquele formato.",
    how_2_title: "Personalize a aparência",
    how_2_desc: "Ajuste tamanho, margem, cores, formato dos pontos e cantos. Se quiser, adicione o logo da sua marca ao centro.",
    how_3_title: "Teste e baixe em PNG",
    how_3_desc: "Confira a prévia, escaneie com o celular para validar e baixe o arquivo PNG pronto para usar no digital ou impresso.",

    types_title: "Tipos de QR Code disponíveis",
    types_desc: "Cada tipo gera uma estrutura compatível com leitores comuns de QR Code em celulares modernos.",
    type_url: "URL: direciona a pessoa para um site, landing page, formulário, cardápio, perfil social ou arquivo online.",
    type_vcard: "vCard: salva dados de contato diretamente na agenda do celular.",
    type_email: "Email: abre o aplicativo de email com destinatário, assunto e mensagem já preenchidos.",
    type_wifi: "Wi-Fi: facilita a conexão a uma rede com SSID, senha e tipo de criptografia.",
    type_sms: "SMS: abre uma mensagem de texto com telefone e conteúdo definidos.",
    type_pix: "PIX: gera um QR Code estático para recebimento de pagamentos e transferências.",
    type_text: "Texto: armazena uma informação simples que aparece diretamente ao escanear.",

    faq_title: "Perguntas frequentes",
    faq_1_q: "Posso adicionar meu logo ao QR Code?",
    faq_1_a: "Sim. Use a opção de logo para enviar uma imagem e posicioná-la no centro. Para manter a leitura confiável, prefira logos simples e teste o QR Code antes de imprimir.",
    faq_3_q: "O QR Code continua funcionando com cores e formatos diferentes?",
    faq_3_a: "Sim, desde que exista contraste suficiente entre os pontos e o fundo. Evite cores muito claras nos pontos e sempre teste a leitura no celular.",
    faq_4_q: "O download é feito em qual formato?",
    faq_4_a: "Os arquivos são gerados no formato PNG de alta qualidade, ideal para uso digital e impressão.",

    see1: "Copiar e Colar Emojis",
    see2: "Gerador de Código de Barras",
    see3: "Cores Aleatórias",
    see4: "Cores Dominantes da Imagem",
  },
  es: {
    see1: "Copiar y Pegar Emojis",
    see2: "Generador de Código de Barras",
    see3: "Colores Aleatorios",
    see4: "Colores Dominantes de la Imagen",
  },
  fr: {
    see1: "Copier Coller Emoji",
    see2: "Générateur de Code-Barres",
    see3: "Couleurs Aléatoires",
    see4: "Couleurs Dominantes de l'Image",
  },
  it: {
    see1: "Copia e Incolla Emoji",
    see2: "Generatore di Codice a Barre",
    see3: "Colori Casuali",
    see4: "Colori Dominanti dell'Immagine",
  },
  id: {
    see1: "Pemilik Emoji",
    see2: "Pembuat Barcode",
    see3: "Warna Acak",
    see4: "Warna Dominan Gambar",
  }
}
</i18n>
