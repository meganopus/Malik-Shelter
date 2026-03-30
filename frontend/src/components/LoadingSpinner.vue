<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { DotLottieVue } from '@lottiefiles/dotlottie-vue'

const visible = ref(false)
onMounted(() => {
  setTimeout(() => { visible.value = true }, 2000)
})
</script>

<template>
  <div class="flex flex-col items-center justify-center py-24 min-h-[400px]">
    <Transition name="fade">
      <DotLottieVue
        v-if="visible"
        src="/loading-cat.lottie"
        autoplay
        loop
        style="width: 300px; height: 300px;"
      />
    </Transition>
    <p v-if="visible" class="mt-6 text-xl text-muted">Loading adorable pets...</p>
  </div>
</template>

<style scoped>
.fade-enter-active {
  transition: opacity 0.4s ease;
}
.fade-enter-from {
  opacity: 0;
}
</style>
