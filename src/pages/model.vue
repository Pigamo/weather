<template>
  <!-- Use it like any other HTML element -->
  <model-viewer
    class="w-screen h-screen"
    alt="Modem"
    src="./modem.glb"
    ar
    ar-modes="webxr scene-viewer quick-look"
    skybox-image="./lebombo_1k.hdr"
    field-of-view="5deg"
    camera-orbit="110deg 70deg "
    shadow-intensity="1"
    camera-controls
    enable-pan
  ></model-viewer>
</template>

<script setup lang="ts"></script>

<style scoped></style>
