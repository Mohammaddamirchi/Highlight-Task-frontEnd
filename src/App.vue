<template>
  <main class="main" style="direction: rtl;">
    <div class="container">
      <section class="hero">
        <h1 class="heroTitle">
          <span>تبدیل صدا به متن</span>
        </h1>

        <p class="heroSub">
          فایل صوتی خود را آپلود کنید تا متن و تحلیل مکالمه با هوش مصنوعی دریافت کنید
        </p>

        <div class="cardWrap">
          <div class="card">
            <p class="cardTitle">آپلود فایل صوتی</p>

            <div
              class="dropzone"
              :class="{ dropzoneActive: isDragging }"
              @dragenter.prevent="isDragging = true"
              @dragover.prevent="isDragging = true"
              @dragleave.prevent="isDragging = false"
              @drop.prevent="onDrop"
            >
              <input
                ref="fileInputEl"
                type="file"
                class="srOnly"
                multiple
                accept=".wav"
                @change="onPickFilesAndSubmit"
              />

              <button
                class="btn btnPrimary"
                type="button"
                :disabled="loading"
                @click="openFileDialog"
              >
                {{ loading ? 'در حال پردازش...' : 'انتخاب فایل' }}
              </button>

              <div class="smallText">
                <div><strong>فرمت مجاز:</strong> wav</div>

                <div v-if="pickedFiles.length" style="margin-top: 8px">
                  <strong>فایل‌ها:</strong>
                  {{ pickedFiles.map(f => f.name).join(', ') }}
                </div>
              </div>
            </div>

            <!-- ERROR -->
            <div v-if="error" class="errorBox">
              ❌ {{ error }}
            </div>

            <!-- TRANSCRIPT -->
            <div v-if="result?.transcript" class="resultBox">
              <h3>متن مکالمه</h3>
              <pre>{{ result.transcript }}</pre>
            </div>

            <!-- ANALYSIS -->
            <div v-if="result?.data" class="resultBox">
              <h3>تحلیل مکالمه</h3>

              <p><strong>امتیاز:</strong> {{ result.data.score }}/5</p>

              <p><strong>خلاصه:</strong></p>
              <p>{{ result.data.summary }}</p>

              <p><strong>نقاط قوت:</strong></p>
              <ul>
                <li v-for="(s, i) in result.data.strengths" :key="i">
                  {{ s }}
                </li>
              </ul>

              <p><strong>نقاط ضعف:</strong></p>
              <ul>
                <li v-for="(w, i) in result.data.weaknesses" :key="i">
                  {{ w }}
                </li>
              </ul>
            </div>
          </div>
        </div>
      </section>
    </div>
  </main>
</template>

<script setup lang="ts">
import axios from 'axios'
import { ref } from 'vue'

const fileInputEl = ref<HTMLInputElement | null>(null)
const pickedFiles = ref<File[]>([])
const loading = ref(false)
const error = ref('')
const result = ref<any>(null)
const isDragging = ref(false)

function openFileDialog() {
  fileInputEl.value?.click()
}

function onDrop(e: DragEvent) {
  isDragging.value = false
  const files = Array.from(e.dataTransfer?.files ?? [])
  if (files.length) {
    pickedFiles.value = files
    submitFiles()
  }
}

function onPickFilesAndSubmit(e: Event) {
  const input = e.target as HTMLInputElement
  pickedFiles.value = Array.from(input.files ?? [])
  if (pickedFiles.value.length) {
    submitFiles()
  }
}

async function submitFiles() {
  loading.value = true
  error.value = ''
  result.value = null

  const formData = new FormData()
  pickedFiles.value.forEach(file => {
    formData.append('files', file)
  })

  try {
    const res = await axios.post(
      'http://localhost:5000/upload_folder',
      formData,
      { headers: { 'Content-Type': 'multipart/form-data' } }
    )

    if (!res.data.success) {
      throw new Error(res.data.error || 'خطای ناشناخته')
    }

    result.value = res.data
  } catch (err: any) {
    error.value =
      err?.response?.data?.error ||
      err?.message ||
      'خطا در ارتباط با سرور'
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.srOnly {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
}

.dropzone {
  border: 2px dashed #cbd5e1;
  padding: 24px;
  border-radius: 12px;
  text-align: center;
}

.dropzoneActive {
  border-color: #6366f1;
  background: #eef2ff;
}

.btnPrimary {
  padding: 10px 18px;
  border-radius: 8px;
}

.resultBox {
  margin-top: 20px;
  background: #f8fafc;
  padding: 16px;
  border-radius: 10px;
}

.errorBox {
  margin-top: 16px;
  color: #842029;
  background: #f8d7da;
  padding: 10px;
  border-radius: 8px;
}

pre {
  white-space: pre-wrap;
}
</style>
