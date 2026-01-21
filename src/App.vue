<script setup lang="ts">
import { useDevicesList, useUserMedia } from "@vueuse/core";
import { reactive, shallowRef, useTemplateRef, watchEffect } from "vue";

const currentCamera = shallowRef<string>();

const { videoInputs: cameras } = useDevicesList({
  requestPermissions: true,
  onUpdated() {
    if (!cameras.value.find((i) => i.deviceId === currentCamera.value))
      currentCamera.value = cameras.value[0]?.deviceId;
  },
});

const video = useTemplateRef("video");

const { stream, enabled, start } = useUserMedia({
  constraints: reactive({ video: { deviceId: { exact: currentCamera } } }),
});

watchEffect(() => {
  if (video.value) video.value.srcObject = stream.value!;
});

start();
</script>

<template>
  <div class="flex flex-col gap-2 items-center my-8 px-8">
    <button @click="enabled = !enabled">
      {{ enabled ? "Arrêter" : "Commencer" }}
    </button>

    <div
      v-for="camera of cameras"
      :key="camera.deviceId"
      class="cursor-pointer"
      :class="{ 'font-bold': currentCamera === camera.deviceId }"
      @click="currentCamera = camera.deviceId"
    >
      {{ camera.label }}
    </div>

    <video
      ref="video"
      muted
      autoplay
      class="aspect-9/16 rounded-xl border max-h-[80vh]"
    />
  </div>
</template>

<style scoped></style>
