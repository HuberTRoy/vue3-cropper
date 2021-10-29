<script setup lang="ts">
  // This starter template is using Vue 3 <script setup> SFCs
  // Check out https://v3.vuejs.org/api/sfc-script-setup.html#sfc-script-setup
  import VueCropper from "./components/vue-cropper.vue";

  // const print = (x) => console.log(x)
  // import HelloWorld from './components/HelloWorld.vue'
</script>

<template>
  <HelloWorld />
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
