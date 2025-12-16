<template>
  <main class="main" style="direction: rtl;">
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
            </div>

            <p class="cardTitle" v-if="activeTab === 'upload'">فایل‌های صوتی را از دستگاه بارگذاری کنید</p>

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

    <div v-if="error" class="text-red-500 mt-4">{{ error }}</div>

    <div v-if="result" class="mt-6">
      <h2 class="text-xl font-semibold mb-2">Transcript</h2>
      <pre class="bg-gray-100 p-3 rounded overflow-x-auto">{{ result.transcript }}</pre>

      <h2 class="text-xl font-semibold mt-4 mb-2">Resolution</h2>
      <pre>{{ result.resolution }}</pre>

      <h2 class="text-xl font-semibold mt-4 mb-2">Analysis</h2>
      <pre>{{ result.analysis }}</pre>

      <h2 class="text-xl font-semibold mt-4 mb-2">Action</h2>
      <pre>{{ result.action }}</pre>

      <h2 class="text-xl font-semibold mt-4 mb-2">Extra Insights</h2>
      <pre>{{ result.extra }}</pre>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";

const selectedFiles = ref([]);
const uploading = ref(false);
const result = ref(null);
const error = ref("");

function handleFiles(event) {
  selectedFiles.value = Array.from(event.target.files);
  result.value = null;
  error.value = "";
}

async function uploadFiles() {
  if (!selectedFiles.value.length) return;

  const formData = new FormData();
  selectedFiles.value.forEach((file) => {
    formData.append("files", file);
  });

  uploading.value = true;
  error.value = "";
  result.value = null;

  try {
    const response = await axios.post("http://localhost:5000/upload_folder", formData, {
      headers: {
        "Content-Type": "multipart/form-data",
      },
    });

    result.value = response.data;
  } catch (err) {
    console.error(err);
    error.value = err.response?.data?.error || "Upload failed";
  } finally {
    uploading.value = false;
  }
}
</script>

<style>
/* فقط برای کمی استایل بهتر */
.container {
  max-width: 700px;
}
</style>
