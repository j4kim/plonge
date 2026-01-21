<script setup lang="ts">
import { useDevicesList, useUserMedia } from "@vueuse/core";
import { reactive, shallowRef, useTemplateRef, watch, watchEffect } from "vue";

const currentCamera = shallowRef<string>();

const { videoInputs: cameras } = useDevicesList({
  requestPermissions: true,
  constraints: { audio: false },
  onUpdated() {
    if (!cameras.value.find((i) => i.deviceId === currentCamera.value))
      currentCamera.value = cameras.value[0]?.deviceId;
  },
});

const video = useTemplateRef("video");

const constraints = reactive({
  video: { deviceId: { exact: currentCamera }, aspectRatio: { exact: 9 / 16 } },
  audio: false,
});

const { stream, enabled, start } = useUserMedia({ constraints });

watchEffect(() => {
  if (video.value) video.value.srcObject = stream.value!;
});

start();
</script>

<template>
  <div class="w-screen h-screen bg-black relative">
    <video ref="video" muted autoplay class="w-full h-full" />
    <div
      class="text-white py-[3vw] absolute top-0 w-full h-full flex flex-col items-center justify-between"
    >
      <h1
        class="text-[5vw] px-[3vw] uppercase bg-black/10 backdrop-blur-sm rounded-full font-bold"
      >
        Filmez ce que vous voulez !
      </h1>
      <div
        class="relative bottom-0 w-full text-[3vw] flex justify-between px-[4vw]"
      >
        <select
          v-model="currentCamera"
          class="px-[2vw] py-[1vw] max-w-1/3 bg-black/10 backdrop-blur-sm rounded-full"
        >
          <option
            v-for="camera of cameras"
            :key="camera.deviceId"
            :value="camera.deviceId"
          >
            {{ camera.label }}
          </option>
        </select>
        <button
          class="px-[2vw] py-[1vw] max-w-1/3 bg-black/10 backdrop-blur-sm rounded-full"
          @click="enabled = !enabled"
        >
          {{ enabled ? "Arrêter" : "Commencer" }}
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped></style>
