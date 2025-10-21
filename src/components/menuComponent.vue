<script setup lang="ts">
import loginComponent from './loginComponent.vue'
import { ref, onMounted } from 'vue'
const isLoggedIn = ref(false)
onMounted(() => {
  const accessToken = window.localStorage.getItem('access_token')
  if (accessToken) {
    isLoggedIn.value = true
  }
})
</script>

<template>
  <div role="banner" class="bg-neutral-900 min-h-screen w-full">
    <nav class="flex items-center justify-between p-4">
      <div
        class="flex-1 font-bold text-4xl m-3.5 bg-linear-to-r from-green-500 to-slate-100 bg-clip-text text-transparent"
      >
        <a href=""> Moodify </a>
      </div>
      <ul class="md:flex flex-8 justify-end-safe content space-x-4">
        <li class="self-center">
          <a
            href="#/about"
            class="bg-linear-to-b from-green-400 to-slate-400 bg-clip-text text-transparent text-lg hover:text-gray-300 transition-colors duration-300"
            >About</a
          >
        </li>
        <li class="self-center">
          <a
            v-if="isLoggedIn"
            href="#/profile"
            class="bg-linear-to-b from-green-400 to-slate-400 bg-clip-text text-transparent text-lg hover:text-gray-300 transition-colors duration-300"
            >Profile</a
          >
        </li>
        <login-component v-if="!isLoggedIn" />
        <logout-button v-else />
      </ul>
    </nav>
  </div>
</template>

<style scoped>
.min-h-screen {
  min-height: 5vh;
}
</style>
