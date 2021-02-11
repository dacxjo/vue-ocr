<template>
  <main class="h-screen w-full bg-gray-50 overflow-hidden">
    <header class="w-full h-16 flex items-center justify-center text-center">
      <h1 class="text-2xl text-gray-800 src-sans-pro">OCR.Space Test</h1>
    </header>
    <hr />

    <section class="container mx-auto mt-10 h-64">
      <div class="grid grid-cols-2 h-full">
        <div class="p-10">
          <h2 class="src-sans-pro text-gray-700">Select the file</h2>
          <form
            ref="form"
            @submit.prevent="readDocument"
            class="mt-5 flex flex-col items-start"
          >
            <input
              @change="fileSelected"
              type="file"
              name="file"
              id=""
              class="src-sans-pro"
            />
            <iframe
              v-if="selectedFile"
              :src="selectedFile"
              frameborder="0"
              class="w-full h-64 mt-5"
            ></iframe>
            <div class="flex items-center justify-center">
              <button
                :disabled="isReading || !selectedFile"
                class="focus:outline-none disabled:opacity-50 text-sm w-24 py-3 rounded-md font-semibold text-white bg-gray-900 ring-2 ring-gray-900 mt-5 src-sans-pro"
                type="submit"
              >
                Read File
              </button>
              <progress
                v-show="isReading"
                class="h-10 w-full ml-5 mt-5 rounded-lg"
                max="100"
                :value="percentReaded"
              >
                {{ percentReaded }}%
              </progress>
            </div>
          </form>
        </div>
        <div class="border-l border-gray-800 p-10 overflow-scroll">
          <h2 class="src-sans-pro text-gray-700">Document data</h2>
          <a
            :href="searchableURL"
            v-if="searchableURL.length > 0"
            target="_blank"
            class="focus:outline-none block w-32 text-sm px-3  py-3 rounded-md font-semibold text-white bg-gray-900 ring-2 ring-gray-900 mt-5 src-sans-pro"
            type="submit"
          >
            View in new tab
          </a>
          <iframe
            class="w-full h-64 mt-5"
            v-if="searchableURL.length > 0"
            :src="searchableURL"
            frameborder="0"
          ></iframe>
          <textarea
            readonly
            v-model="parsedText"
            class="w-full h-full mt-5"
            name=""
            id=""
            cols="30"
            rows="10"
          ></textarea>
        </div>
      </div>
    </section>
    <footer
      class="h-10 w-full absolute bottom-0 left-0 bg-gray-100 flex items-center justify-center"
    >
      <p class="src-sans-pro text-gray-900 text-center">Arpen Technologies ©</p>
    </footer>
  </main>
</template>

<script setup>
import axios from "axios";
import { ref } from "vue";

const form = ref("form");
const selectedFile = ref(null);
const parsedText = ref("");
const searchableURL = ref("");
const isReading = ref(false);
const percentReaded = ref(0);

// eslint-disable-next-line no-unused-vars
function fileSelected(event) {
  const file = event.target.files[0];
  const fr = new FileReader();
  fr.onload = event => {
    selectedFile.value = event.target.result;
  };
  fr.readAsDataURL(file);
}

// eslint-disable-next-line no-unused-vars
async function readDocument() {
  const fd = new FormData(form.value);
  fd.append("language", "spa");
  fd.append("apikey", "ccd56ab63a88957");
  fd.append("isOverlayRequired", true);
  fd.append("isTable", true);
  fd.append("isCreateSearchablePdf", true);
  fd.append("OCREngine", 1);

  isReading.value = true;

  const config = {
    onUploadProgress: function(progressEvent) {
      const percentCompleted = Math.round(
        (progressEvent.loaded * 100) / progressEvent.total
      );
      percentReaded.value = percentCompleted;
    }
  };

  const endpoint = `${location.protocol}//api.ocr.space/parse/image`;

  await axios.post(endpoint, fd, config).then(({ data }) => {
    const { ParsedResults, SearchablePDFURL } = data;
    parsedText.value = ParsedResults[0].ParsedText;
    searchableURL.value = SearchablePDFURL;
    isReading.value = false;
  });
}
</script>

<style></style>
