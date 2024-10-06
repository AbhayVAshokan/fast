<script setup>
const isCalculating = ref(false);
const isFinished = ref(false);

// Ignore the first 200ms of download as buffer.
const isBufferTime = ref(false);
const bufferTimeDownload = ref([0, 0, 0, 0, 0]);

const bytesDownloaded = ref([0, 0, 0, 0, 0]);
const startTime = ref(null);

const formatSpeed = (bitsPerSecond) => {
  const kbps = bitsPerSecond / 1_000;
  const mbps = bitsPerSecond / 1_000_000;

  if (mbps >= 10) return { speed: mbps, unit: "M" };
  if (kbps >= 10) return { speed: kbps, unit: "K" };
  return { speed: bitsPerSecond, unit: "" };
};

const downloadSpeed = computed(() => {
  if (!startTime.value || isBufferTime.value)
    return { speed: null, unit: null };

  const elapsedTimeMs = Date.now() - startTime.value - 200;
  const totalBytes = bytesDownloaded.value.reduce(
    (sum, bytes, index) => sum + bytes - bufferTimeDownload.value[index]
  );
  const bitsPerSecond = (totalBytes * 8) / (elapsedTimeMs / 1_000);

  return formatSpeed(bitsPerSecond);
});

const fetchFile = (url, index) => {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.addEventListener("progress", (e) => {
      bytesDownloaded.value[index] = e.loaded;
      if (isBufferTime.value) bufferTimeDownload.value[index] = e.loaded;
    });

    xhr.addEventListener("loadend", resolve);
    xhr.addEventListener("abort", resolve);
    xhr.addEventListener("error", reject);
    xhr.open("GET", url);
    xhr.setRequestHeader("Cache-Control", "no-cache");
    xhr.send();
  });
};

const calculateDownloadSpeed = async () => {
  isBufferTime.value = true;
  isCalculating.value = true;
  isFinished.value = false;
  startTime.value = new Date();
  const urls = [
    "/assets/sample.jpg?step=1",
    "/assets/sample.jpg?step=2",
    "/assets/sample.jpg?step=3",
    "/assets/sample.jpg?step=4",
    "/assets/sample.jpg?step=5",
  ];
  const requests = urls.map((url, index) => fetchFile(url, index));

  const bufferTimeout = setTimeout(() => {
    isBufferTime.value = false;
    clearTimeout(bufferTimeout);
  }, 250);

  const timeout = setTimeout(
    () => requests.forEach((req) => req.abort()),
    20_000
  );
  await Promise.all(requests);
  clearTimeout(timeout);
  isFinished.value = true;
};

useHead({
  htmlAttrs: { lang: "en" },
  link: [{ rel: "icon", type: "image/png", href: "/favicon.png" }],
});

useSeoMeta({
  title: "Fast | Internet speed test",
  ogTitle: "Fast | Internet speed test",
  twitterTitle: "Fast | Internet speed test",
  description: "Check your internet speed using this simple web app.",
  ogDescription: "Check your internet speed using this simple web app.",
  twitterDescription: "Check your internet speed using this simple web app.",
  twitterCreatorId: "@abhayvashokan",
  twitterSite: "@abhayvashokan",
  ogUrl: "https://fast.abhay.app",
  keywords: [
    "speedtest",
    "speed test",
    "bandwidth",
    "ping",
    "throughput",
    "nuxt",
    "nuxt3",
    "vue3",
  ],
  ogImage: "https://fast.abhay.app/cover.png",
  twitterImage: "https://fast.abhay.app/cover.png",
  twitterCard: "summary_large_image",
});
</script>

<template>
  <main class="max-w-6xl mx-auto p-8 flex flex-col h-screen relative">
    <NuxtRouteAnnouncer />
    <Button
      @calculate-download-speed="calculateDownloadSpeed"
      :is-calculating="isCalculating"
      :is-finished="isFinished"
    >
      <p v-if="!isCalculating" class="text-5xl">Go</p>
      <div v-else class="text-8xl md:text-9xl flex items-end justify-center">
        <p>{{ Math.round(downloadSpeed.speed) }}</p>
        <p class="text-3xl md:text-5xl leading-normal">
          {{ downloadSpeed.unit }}bps
        </p>
      </div>
    </Button>
    <copyright></copyright>
  </main>
</template>

<style>
:root {
  --background: #c9e4ca;
  --primary-button-foreground: #1e212b;
}

body {
  font-family: Orbitron, system-ui, -apple-system, BlinkMacSystemFont,
    "Segoe UI", Roboto, Oxygen, Ubuntu, Cantarell, "Open Sans", "Helvetica Neue",
    sans-serif;
  background-color: var(--background);
  overflow: hidden;
}
</style>
