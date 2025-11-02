<script setup lang="ts">
import { ref, onMounted } from 'vue'
import axios from 'axios'

const isLoggedIn = ref(false)
const userInput = ref('')

const sendPrompt = async () => {
  if (!isLoggedIn.value) return
  const payload = userInput.value.trim()
  if (!payload) return
  try {
    console.log('Send request:', payload)
    await axios.post(`http://127.0.0.1:8000/prompt/${payload}`, payload, {})
    userInput.value = ''
  } catch (err) {
    console.error('Failed to send request:', err)
  }
}
onMounted(() => {
  const accessToken = window.localStorage.getItem('access_token')
  if (accessToken) {
    isLoggedIn.value = true
  }
})
</script>
<template>
  <div class="flex p-6 bg-neutral-900 min-h-screen text-white justify-start items-center flex-col">
    <h1 class="text-2xl font-bold mb-4">Welcome to Moodify</h1>
    <div v-if="isLoggedIn">
      <p class="text-green-600">You are logged in!</p>
    </div>
    <div v-else>
      <p class="text-red-600">Please log in to continue.</p>
    </div>
    <div class="w-full max-w-3xl mt-5">
      <div class="border border-neutral-800 rounded-2xl bg-neutral-800/60 backdrop-blur p-3">
        <div class="flex gap-2 items-end">
          <textarea
            id="textarea"
            v-model="userInput"
            :disabled="!isLoggedIn"
            class="flex-1 bg-transparent outline-none text-base resize-none max-h-48 min-h-12 p-3 rounded-xl border-2 border-green-400 placeholder-neutral-400 disabled:opacity-50 disabled-cursor-not-allowed overflow-hidden focus:border-gradient-to-r focus:from-green-400 focus:to-slate-400 transition-colors duration-300"
            placeholder="How are you feeling today? Tell me more..."
            rows="4"
            @keydown.enter.exact.prevent="isLoggedIn && userInput.trim()"
          />
          <button
            id="button"
            type="button"
            :disabled="!isLoggedIn || !userInput.trim()"
            class="px-4 py-2 rounded-xl text-neutral-50 bg-green-500 hover:bg-green-600 disabled:bg-neutral-700 disabled:text-neutral-400 transition-colors duration-300"
            @click="isLoggedIn && userInput.trim() && (sendPrompt(), (userInput = ''))"
            title="Send"
          >
            Send
          </button>
        </div>
        <div class="text-xs text-neutral-400 mt-2 px-1">
          Press Enter to send • Shift+Enter for newline
        </div>
      </div>
      <iframe
        style="border-radius: 12px ; margin-top: 20px;"
        src="https://open.spotify.com/embed/track/3eE2OyYKMf3YIFS1e74ulg?utm_source=generator"
        width="100%"
        height="352"
        frameBorder="0"
        allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture"
        loading="lazy"
      ></iframe>
    </div>
  </div>
</template>
<style scoped>
.min-h-screen {
  min-height: 92.5vh;
}
</style>
