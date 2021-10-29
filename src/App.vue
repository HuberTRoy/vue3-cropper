<script setup lang="ts">
  import VueCropper from "./components/vue-cropper.vue";
</script>

<template>
  <VueCropper :image="'./src/assets/gouzi.jpeg'" />
</template>

<style>
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
</style>
