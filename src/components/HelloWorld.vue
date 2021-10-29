<script setup lang="ts">
import { ref, customRef } from 'vue'

defineProps<{ msg: string }>()

let myRef = (val:any) => {
  let p = ref(val)
  const setP = (newVal:any) => {
    p.value = newVal

    return p.value
  }
  return [p.value, setP, p]
}

function useMyRef(value, delay = 200) {
  return customRef((track, trigger) => {
    return {
      get() {
        track()
        return value
      },
      set(newValue) {
        value = newValue
        trigger()
      }
    }
  })
}

let count = useMyRef(0)

const countPlus = () => {
  // that.count++
  // count += 1
  // console.log(count)
  count.value++
  // console.log(cc)

}

</script>

<template>
  <h1>{{ msg }}</h1>

  <p>
    Recommended IDE setup:
    <a href="https://code.visualstudio.com/" target="_blank">VSCode</a>
    +
    <a href="https://github.com/johnsoncodehk/volar" target="_blank">Volar</a>
  </p>

  <p>See <code>README.md</code> for more information.</p>

  <p>
    <a href="https://vitejs.dev/guide/features.html" target="_blank">
      Vite Docs
    </a>
    |
    <a href="https://v3.vuejs.org/" target="_blank">Vue {{ cc }} Docs</a>
  </p>

  <button type="button" @click="countPlus">count is: {{ count }}</button>
  <p>
    Edit
    <code>components/HelloWorld.vue</code> to test hot module replacement.
  </p>
</template>

<style scoped>
a {
  color: #42b983;
}

label {
  margin: 0 0.5em;
  font-weight: bold;
}

code {
  background-color: #eee;
  padding: 2px 4px;
  border-radius: 4px;
  color: #304455;
}
</style>
