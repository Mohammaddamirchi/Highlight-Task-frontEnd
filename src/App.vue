<template>
  <div class="container mx-auto p-4">
    <h1 class="text-2xl font-bold mb-4">Upload Conversation WAVs</h1>

    <input type="file" multiple @change="handleFiles" accept=".wav" class="mb-4" />

    <button
      @click="uploadFiles"
      :disabled="!selectedFiles.length || uploading"
      class="bg-blue-500 text-white px-4 py-2 rounded disabled:opacity-50"
    >
      {{ uploading ? "Uploading..." : "Upload & Transcribe" }}
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
const rm -rf node_modules package-lock.json
npm install
uploading = ref(false);
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
