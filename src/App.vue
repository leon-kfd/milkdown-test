<script setup>
import { ref, nextTick } from 'vue'
import Milk from './components/Milk.vue'
const enableTooltip = ref(false)
const enableSlash = ref(false)
const enableGfm = ref(false)
const enablePrism = ref(false)
const onChange = (val) => {
  console.log(val)
}
const flag = ref(true)
const forceUpdate = async () => {
  flag.value = false
  await nextTick()
  flag.value = true
}
</script>

<template>
  <input type="checkbox" v-model="enableTooltip"> enableTooltip <br>
  <input type="checkbox" v-model="enableSlash"> enableSlash <br>
  <input type="checkbox" v-model="enableGfm"> enableGfm <br>
  <input type="checkbox" v-model="enablePrism"> enablePrism <br>

  <button @click="forceUpdate">Refresh</button>

  <component
    v-if="flag"
    :is="Milk" 
    style="wdith: 800px;height: 600px;box-shadow: 0 0 8px #ccc" 
    :fontSize="24"
    textColor="#b22"
    :enableTooltip="enableTooltip"
    :enableSlash="enableSlash"
    :enableGfm="enableGfm"
    :enablePrism="enablePrism"
    markdown="# Hello world" 
    @change="onChange"/>
</template>

<style>
</style>
