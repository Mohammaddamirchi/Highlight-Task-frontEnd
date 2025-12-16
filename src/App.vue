<template>
  <div class="min-h-screen bg-[#f5f7ff]">
    <header class="flex justify-between items-center px-10 py-6">
      <div class="text-xl font-bold text-indigo-600">Highlight</div>
      <nav class="flex gap-6 text-gray-600">
        <a href="#">امکانات</a>
        <a href="#">قیمت‌گذاری</a>
        <a href="#">راهنما</a>
        <a href="#">وبلاگ</a>
      </nav>
    </header>

    <section class="text-center mt-16 px-4">
      <h1 class="text-4xl font-extrabold text-indigo-600 mb-4">
        تبدیل مکالمه صوتی به متن
      </h1>
      <p class="max-w-2xl mx-auto text-gray-600 text-lg">
        فایل‌های صوتی مکالمه را آپلود کنید تا متن کامل، تحلیل تماس و پیشنهاد اقدام بعدی را دریافت کنید.
      </p>

      <div class="flex justify-center gap-10 mt-10 text-gray-600">
        <div>🎧 پشتیبانی از WAV</div>
        <div>🌍 زبان فارسی</div>
        <div>📄 خروجی تحلیلی</div>
      </div>
    </section>

    <section class="mt-14 flex justify-center px-4">
      <div class="bg-white w-full max-w-xl rounded-xl shadow-lg p-6">
        <div class="flex border-b mb-4">
          <button class="font-semibold text-indigo-600 border-b-2 border-indigo-600 pb-2">
            آپلود فایل
          </button>
        </div>

        <div
          class="border-2 border-dashed border-indigo-300 rounded-lg p-8 text-center cursor-pointer hover:bg-indigo-50 transition"
          @click="fileInput.click()"
        >
          <p class="text-gray-600 mb-2">فایل‌های WAV مکالمه را اینجا رها کنید</p>
          <p class="text-sm text-gray-400 mb-4">یا</p>
          <button class="bg-indigo-600 text-white px-5 py-2 rounded">
            انتخاب فایل
          </button>
          <input
            type="file"
            ref="fileInput"
            class="hidden"
            multiple
            accept=".wav"
            @change="handleFiles"
          />
        </div>

        <div v-if="files.length" class="mt-4 text-sm text-gray-600">
          <p class="font-semibold mb-1">فایل‌های انتخاب‌شده:</p>
          <ul class="list-disc list-inside">
            <li v-for="f in files" :key="f.name">{{ f.name }}</li>
          </ul>
        </div>

        <button
          @click="upload"
          :disabled="uploading"
          class="w-full mt-6 bg-indigo-600 hover:bg-indigo-700 text-white py-3 rounded-lg transition disabled:opacity-50"
        >
          {{ uploading ? "در حال پردازش..." : "شروع پردازش" }}
        </button>

        <p v-if="error" class="text-red-500 text-sm mt-3">{{ error }}</p>
      </div>
    </section>
    
    <section v-if="result" class="max-w-4xl mx-auto mt-16 px-4 space-y-6">
      <ResultCard title="📝 متن مکالمه" :content="result.transcript" />
      <ResultCard title="✅ وضعیت تماس" :content="pretty(result.resolution)" />
      <ResultCard title="📊 تحلیل تماس" :content="pretty(result.analysis)" />
      <ResultCard title="⚡ اقدام پیشنهادی" :content="pretty(result.action)" />
      <ResultCard title="💡 بینش اضافه" :content="pretty(result.extra)" />
    </section>
  </div>
</template>

<script setup>
import { ref, defineComponent } from "vue";
import axios from "axios";

const files = ref([]);
const uploading = ref(false);
const result = ref(null);
const error = ref("");
const fileInput = ref(null);

function handleFiles(e) {
  files.value = Array.from(e.target.files);
  result.value = null;
  error.value = "";
}

async function upload() {
  if (!files.value.length) return;

  const formData = new FormData();
  files.value.forEach(f => formData.append("files", f));

  uploading.value = true;
  error.value = "";

  try {
    const res = await axios.post(
      "http://localhost:5000/upload_folder",
      formData
    );
    result.value = res.data;
  } catch (e) {
    error.value = "خطا در پردازش فایل‌ها";
  } finally {
    uploading.value = false;
  }
}

function pretty(obj) {
  return JSON.stringify(obj, null, 2);
}

const ResultCard = defineComponent({
  props: ["title", "content"],
  template: `
    <div class="bg-white rounded-xl shadow p-5">
      <h3 class="font-bold text-gray-800 mb-3">{{ title }}</h3>
      <pre class="bg-gray-50 p-4 rounded text-sm whitespace-pre-wrap overflow-x-auto">
{{ content }}
      </pre>
    </div>
  `
});
</script>
