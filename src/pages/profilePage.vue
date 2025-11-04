<script setup lang="ts">
import menuComponent from '@/components/menuComponent.vue'
import { ref, onMounted } from 'vue'
const isLoggedIn = ref(false)
type SpotifyUser = {
  id: string
  display_name: string | null
  email?: string
  images?: Array<{ url: string }>
  [key: string]: unknown
}

const userProfile = ref<SpotifyUser | null>(null)
const userProfileLoading = ref(false)
const userProfileError = ref<string | null>(null)

async function fetchSpotifyProfile(): Promise<SpotifyUser | null> {
  const token = window.localStorage.getItem('access_token')
  if (!token) {
    userProfileError.value = 'Missing access token.'
    return null
  }

  userProfileLoading.value = true
  userProfileError.value = null

  try {
    const res = await fetch('https://api.spotify.com/v1/me', {
      headers: { Authorization: `Bearer ${token}` },
    })

    if (!res.ok) {
      const text = await res.text()
      throw new Error(`Spotify API error ${res.status}: ${text || res.statusText}`)
    }

    const data: SpotifyUser = await res.json()
    userProfile.value = data
    return data
  } catch (e) {
    userProfileError.value = e instanceof Error ? e.message : 'Unknown error'
    return null
  } finally {
    userProfileLoading.value = false
  }
}
onMounted(() => {
  const accessToken = window.localStorage.getItem('access_token')
  if (accessToken) {
    isLoggedIn.value = true
    fetchSpotifyProfile()
  }
})
</script>
<template>
  <menu-component />
  <div class="flex p-6 bg-neutral-900 min-h-screen text-white justify-start items-center flex-col">
    <h1 class="text-2xl font-bold mb-4">Profile Page</h1>
    <div v-if="isLoggedIn">
      <p class="text-green-500">Welcome back!</p>
    </div>
    <div v-if="userProfileLoading">Loading...</div>
    <div v-if="userProfileError" class="text-red-500">{{ userProfileError }}</div>
    <div v-if="userProfile">
      <h2 class="text-xl font-bold mt-4">Your Profile</h2>
      <p class="text-md">Name: {{ userProfile.display_name }}</p>
      <p class="text-md">Email: {{ userProfile.email }}</p>
      <!-- <p class="text-md">Picture: {{ userProfile.images[0].url }}</p> -->
    </div>
  </div>
</template>
<style scoped></style>
