<template>
  <main class="main">
    <div class="container">
      <section class="hero">
        <h1 class="heroTitle"><span>تبدیل صدا به متن</span></h1>
        <p class="heroSub">
           به شما امکان می‌دهد فایل‌های صوتی را آپلود کنید یا لینک‌ها را جای‌گذاری کنید و به سرعت آنها را با هوش مصنوعی به متن تبدیل کنید
        </p>

        <div class="cardWrap">
          <div class="card" aria-label="Upload card">
            <div class="tabs" role="tablist" aria-label="Upload modes">
              <button
                class="tabBtn"
                :class="{ tabBtnActive: activeTab === 'upload' }"
                type="button"
                role="tab"
                :aria-selected="activeTab === 'upload'"
                @click="activeTab = 'upload'"
              >
                آپلود فایل
              </button>
              <button
                class="tabBtn"
                :class="{ tabBtnActive: activeTab === 'link' }"
                type="button"
                role="tab"
                :aria-selected="activeTab === 'link'"
                @click="activeTab = 'link'"
              >
                پیست لینک
              </button>
            </div>

            <p class="cardTitle" v-if="activeTab === 'upload'">فایل‌های صوتی را از دستگاه بارگذاری کنید</p>
            <p class="cardTitle" v-else>Paste a YouTube / public media link to transcribe</p>

            <section v-if="activeTab === 'upload'" role="tabpanel">
              <div
                class="dropzone"
                :class="{ dropzoneActive: isDragging }"
                @dragenter.prevent="onDragEnter"
                @dragover.prevent="onDragOver"
                @dragleave.prevent="onDragLeave"
                @drop.prevent="onDrop"
              >

                <input
                  ref="fileInputEl"
                  type="file"
                  class="srOnly"
                  multiple
                  :accept="acceptedMime"
                  @change="onPickFilesAndSubmit"
                />

                <button class="btn btnPrimary" type="button" @click="openFileDialog" :disabled="loading">
                  {{ loading ? 'در حال پردازش...' : 'انتخاب فایل' }}
                </button>

                <div class="smallText">
                  <div dir="rtl"><strong>فرمت قابل قبول:</strong> wav </div>

                  <div v-if="pickedFiles.length" style="margin-top: 10px; color: #334155">
                    <strong>فایل بارگذاری شده: </strong>
                    <span>{{ pickedFiles.map((f) => f.name).join(', ') }}</span>
                  </div>

                  <div v-if="result" style="margin-top: 10px; color: #0f5132">
                    <strong>نتیجه تبدیل: </strong>
                    <pre>{{ result }}</pre>
                  </div>

                  <div v-if="error" style="margin-top: 10px; color: #842029">
                    <strong>خطا: </strong>{{ error }}
                  </div>
                </div>
              </div>
            </section>


            <section v-else role="tabpanel">
              <div class="linkForm">
                <input
                  v-model.trim="link"
                  class="input"
                  placeholder="Paste link here (e.g. YouTube URL)"
                  inputmode="url"
                />
                <button class="btn btnPrimary" type="button" :disabled="!link" @click="submitLink">
                  Transcribe link
                </button>
                <p v-if="linkSubmitted" class="smallText" style="margin: 0">
                  Submitted link: <strong>{{ linkSubmitted }}</strong>
                </p>
              </div>
            </section>
          </div>
        </div>
      </section>

      <section class="section" id="how">
        <h2 class="sectionTitle">چگونه صدا را به متن تبدیل کنیم؟</h2>

        <div class="steps" aria-label="How it works">
          <div class="step">
            <div class="stepIcon" aria-hidden="true"><SvgUpload /></div>
            <h3>آپلود کن</h3>
            <p>یک فایل صوتی را از دستگاه خود آپلود کنید یا لینکی را جایگذاری کنید</p>
          </div>
          <div class="step">
            <div class="stepIcon" aria-hidden="true"><SvgSparkles /></div>
            <h3>ترجمه به متن</h3>
            <p>ما محتوای شما را پردازش می‌کنیم و متن دقیقی را با هوش مصنوعی تولید می‌کنیم</p>
          </div>
        </div>

        <div class="footerSpace"></div>
      </section>
    </div>
  </main>
</template>

<script setup lang="ts">
import axios from 'axios'

import { computed, h, ref } from 'vue'
const loading = ref(false)
const error = ref('')
const result = ref<any | null>(null)

type Tab = 'upload' | 'link'

const activeTab = ref<Tab>('upload')
const isDragging = ref(false)
const fileInputEl = ref<HTMLInputElement | null>(null)
const pickedFiles = ref<File[]>([])

const link = ref('')
const linkSubmitted = ref('')

const acceptedExt = [
  'wav'
]

function openFileDialog() {
  fileInputEl.value?.click()
}

const acceptedMime = computed(() => acceptedExt.map((x) => `.${x}`).join(','))

function onDragEnter() {
  isDragging.value = true
}

function onDragOver() {
  isDragging.value = true
}

function onDragLeave() {
  isDragging.value = false
}

function onDrop(e: DragEvent) {
  isDragging.value = false
  const files = Array.from(e.dataTransfer?.files ?? [])
  if (!files.length) return
  pickedFiles.value = files
}

function onPickFiles(e: Event) {
  const input = e.target as HTMLInputElement
  const files = Array.from(input.files ?? [])
  pickedFiles.value = files
}

function submitLink() {
  linkSubmitted.value = link.value
}

const SvgHeadphones = () =>
  h(
    'svg',
    { width: 22, height: 22, viewBox: '0 0 24 24', fill: 'none', xmlns: 'http://www.w3.org/2000/svg' },
    [
      h('path', {
        d: 'M4 12a8 8 0 0 1 16 0v6a2 2 0 0 1-2 2h-1v-8h3',
        stroke: 'currentColor',
        'stroke-width': 2,
        'stroke-linecap': 'round',
        'stroke-linejoin': 'round'
      }),
      h('path', {
        d: 'M4 12v6a2 2 0 0 0 2 2h1v-8H4',
        stroke: 'currentColor',
        'stroke-width': 2,
        'stroke-linecap': 'round',
        'stroke-linejoin': 'round'
      })
    ]
  )

const SvgTranslate = () =>
  h(
    'svg',
    { width: 22, height: 22, viewBox: '0 0 24 24', fill: 'none', xmlns: 'http://www.w3.org/2000/svg' },
    [
      h('path', {
        d: 'M4 5h7M7.5 5v2c0 4-2.5 7-5.5 9',
        stroke: 'currentColor',
        'stroke-width': 2,
        'stroke-linecap': 'round',
        'stroke-linejoin': 'round'
      }),
      h('path', {
        d: 'M4 12c1.2 1.3 2.8 2.6 5 3.8',
        stroke: 'currentColor',
        'stroke-width': 2,
        'stroke-linecap': 'round',
        'stroke-linejoin': 'round'
      }),
      h('path', {
        d: 'M14 20l4-11 4 11',
        stroke: 'currentColor',
        'stroke-width': 2,
        'stroke-linecap': 'round',
        'stroke-linejoin': 'round'
      }),
      h('path', {
        d: 'M15.5 16h5',
        stroke: 'currentColor',
        'stroke-width': 2,
        'stroke-linecap': 'round',
        'stroke-linejoin': 'round'
      })
    ]
  )

const SvgDownload = () =>
  h(
    'svg',
    { width: 22, height: 22, viewBox: '0 0 24 24', fill: 'none', xmlns: 'http://www.w3.org/2000/svg' },
    [
      h('path', {
        d: 'M12 3v10m0 0 4-4m-4 4-4-4',
        stroke: 'currentColor',
        'stroke-width': 2,
        'stroke-linecap': 'round',
        'stroke-linejoin': 'round'
      }),
      h('path', {
        d: 'M4 17v3h16v-3',
        stroke: 'currentColor',
        'stroke-width': 2,
        'stroke-linecap': 'round',
        'stroke-linejoin': 'round'
      })
    ]
  )

const SvgUpload = () =>
  h(
    'svg',
    { width: 26, height: 26, viewBox: '0 0 24 24', fill: 'none', xmlns: 'http://www.w3.org/2000/svg' },
    [
      h('path', {
        d: 'M12 16V4m0 0 4 4m-4-4-4 4',
        stroke: 'currentColor',
        'stroke-width': 2,
        'stroke-linecap': 'round',
        'stroke-linejoin': 'round'
      }),
      h('path', {
        d: 'M4 20h16',
        stroke: 'currentColor',
        'stroke-width': 2,
        'stroke-linecap': 'round',
        'stroke-linejoin': 'round'
      })
    ]
  )

const SvgSparkles = () =>
  h(
    'svg',
    { width: 26, height: 26, viewBox: '0 0 24 24', fill: 'none', xmlns: 'http://www.w3.org/2000/svg' },
    [
      h('path', {
        d: 'M12 2l1.2 4.2L17.5 8 13.2 9.8 12 14l-1.2-4.2L6.5 8l4.3-1.8L12 2z',
        stroke: 'currentColor',
        'stroke-width': 2,
        'stroke-linejoin': 'round'
      }),
      h('path', {
        d: 'M19 11l.8 2.8L22.6 15l-2.8 1.2L19 19l-.8-2.8L15.4 15l2.8-1.2L19 11z',
        stroke: 'currentColor',
        'stroke-width': 2,
        'stroke-linejoin': 'round'
      }),
      h('path', {
        d: 'M4.5 12.5l.7 2.4L7.6 16l-2.4 1.1-.7 2.4-.7-2.4L1.4 16l2.4-1.1.7-2.4z',
        stroke: 'currentColor',
        'stroke-width': 2,
        'stroke-linejoin': 'round'
      })
    ]
  )

const SvgShare = () =>
  h(
    'svg',
    { width: 26, height: 26, viewBox: '0 0 24 24', fill: 'none', xmlns: 'http://www.w3.org/2000/svg' },
    [
      h('path', {
        d: 'M4 12v7a1 1 0 0 0 1 1h14a1 1 0 0 0 1-1v-7',
        stroke: 'currentColor',
        'stroke-width': 2,
        'stroke-linecap': 'round',
        'stroke-linejoin': 'round'
      }),
      h('path', {
        d: 'M16 6l-4-4-4 4',
        stroke: 'currentColor',
        'stroke-width': 2,
        'stroke-linecap': 'round',
        'stroke-linejoin': 'round'
      }),
      h('path', {
        d: 'M12 2v14',
        stroke: 'currentColor',
        'stroke-width': 2,
        'stroke-linecap': 'round',
        'stroke-linejoin': 'round'
      })
    ]
  )

function onPickFilesAndSubmit(e: Event) {
  onPickFiles(e)
  if (pickedFiles.value.length) {
    submitFiles()
  }
}

async function submitFiles() {
  if (!pickedFiles.value.length) return

  loading.value = true
  error.value = ''
  result.value = null

  const formData = new FormData()
  pickedFiles.value.forEach((f) => {
    formData.append('files', f)
  })

  try {
    const res = await axios.post(
      'http://localhost:5000/upload_folder', // endpoint بک‌اند
      formData,
      { headers: { 'Content-Type': 'multipart/form-data' } }
    )
    result.value = res.data
  } catch (e: any) {
    error.value = e?.response?.data?.error || 'خطا در پردازش فایل‌ها'
  } finally {
    loading.value = false
  }
}


// async function submitFiles() {
//   if (!pickedFiles.value.length) return

//   loading.value = true
//   error.value = ''
//   result.value = null

//   const formData = new FormData()
//   pickedFiles.value.forEach((f) => {
//     formData.append('files', f)
//   })

//   try {
//     const res = await axios.post(
//       'http://localhost:5000/upload_folder',
//       formData,
//       { headers: { 'Content-Type': 'multipart/form-data' } }
//     )
//     result.value = res.data
//   } catch (e: any) {
//     error.value = e?.response?.data?.error || 'خطا در پردازش فایل‌ها'
//   } finally {
//     loading.value = false
//   }
// }

</script>

<style scoped>
.srOnly {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}
</style>
