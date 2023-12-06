<script lang="ts" setup>
import { onMounted, reactive, ref, watch } from 'vue';
import VSwitch from './components/VSwitch.vue';
const model = reactive({
  quiz: false,
  vids: false,
  docs: false,
});
const isRouteAllow = ref(false)
const allowed = ['localhost', 'winguniversity'];
const checkRouteAllow = () => {
  chrome.tabs.query(
    { active: true, lastFocusedWindow: true },
    (tabs) => {
      let url = tabs[0].url;
      // use `url` here inside the callback because it's asynchronous!
      isRouteAllow.value = allowed.some(allow => new RegExp(allow).test(url));
    }
  );
};
checkRouteAllow()
onMounted(() => {
  if (!isRouteAllow.value) return;
  chrome.storage.sync.get(['quiz'], (val) => (model.quiz = val.quiz ?? false));
  chrome.storage.sync.get(['vids'], (val) => (model.vids = val.vids ?? false));
  chrome.storage.sync.get(['docs'], (val) => (model.docs = val.docs ?? false));
});
watch(
  model,
  (val) => {
    if (!isRouteAllow.value) return;
    chrome.storage.sync.set({ quiz: val.quiz });
    chrome.storage.sync.set({ vids: val.vids });
    chrome.storage.sync.set({ docs: val.docs });
  },
  { deep: true }
);
</script>

<template>
  <main
    class="flex justify-center items-center bg-white dark:bg-slate-800 h-screen"
  >
    <section v-if="isRouteAllow" class="flex flex-col">
      <section class="p-2 pb-0">
        <v-switch v-model="model.quiz" title="Quiz"></v-switch>
      </section>
      <section class="p-2 pb-0">
        <v-switch v-model="model.vids" title="Video"></v-switch>
      </section>
      <section class="p-2 pb-0">
        <v-switch v-model="model.docs" title="Docs"></v-switch>
      </section>
    </section>
    <section class="w-80" v-else>
      <div class="text-center">
        <h1
          class="mt-4 text-3xl font-bold tracking-tight text-gray-900 dark:text-gray-100 sm:text-5xl"
        >
          Our extension only works on
        </h1>
        <a
          class="mt-6 text-base leading-7 text-blue-500 underline"
          target="_blank"
          href="https://winguniversity.wingbank.com.kh/"
        >
          Wing University
        </a>
      </div>
    </section>
  </main>
</template>

<style scoped></style>
