<script setup>
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
  if (!startTime.value) return null;

  const elapsedTimeMs = Date.now() - startTime.value;
  const totalBytes = bytesDownloaded.value.reduce((sum, bytes) => sum + bytes);
  const bitsPerSecond = (totalBytes * 8) / (elapsedTimeMs / 1_000);

  return formatSpeed(bitsPerSecond);
});

const fetchFile = (url, index) => {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.addEventListener(
      "progress",
      (e) => (bytesDownloaded.value[index] = e.loaded)
    );
    xhr.addEventListener("loadend", resolve);
    xhr.addEventListener("abort", resolve);
    xhr.addEventListener("error", reject);
    xhr.open("GET", url);
    xhr.setRequestHeader("Cache-Control", "no-cache");
    xhr.send();
  });
};

const calculateDownloadSpeed = async () => {
  startTime.value = new Date();
  const urls = [
    "/assets/sample-1.jpg",
    "/assets/sample-2.jpg",
    "/assets/sample-3.jpg",
    "/assets/sample-4.jpg",
    "/assets/sample-5.jpg",
  ];
  const requests = urls.map((url, index) => fetchFile(url, index));
  const timeout = setTimeout(
    () => requests.forEach((req) => req.abort()),
    20_000
  );
  await Promise.all(requests);
  clearTimeout(timeout);
};
</script>

<template>
  <div>
    <NuxtRouteAnnouncer />
    <button @click="calculateDownloadSpeed">Start</button>
    <div v-if="downloadSpeed">
      <p>{{ Math.round(downloadSpeed.speed) }} {{ downloadSpeed.unit }}bps</p>
    </div>
  </div>
</template>
