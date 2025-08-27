<template>
  <button class="install" @click="install" :class="{ hidden: !showBtn }">Install</button>
</template>

<script setup>
import { ref, onMounted } from 'vue'
let deferredPrompt = null

const showBtn = ref(false)

onMounted(() => {
  window.addEventListener('beforeinstallprompt', (e) => {
    e.preventDefault()
    deferredPrompt = e
    showBtn.value = true
  })

  window.addEventListener('appinstalled', (e) => {
    showBtn.value = false
  })
})

const install = async () => {
  deferredPrompt.prompt()
  const userChoice = await deferredPrompt.userChoice
  if (userChoice == 'accepted') {
    showBtn.value = false
  }
  deferredPrompt = null
}
</script>

<style scoped>
button {
  cursor: pointer;
}
.hidden {
  display: none;
}
</style>
