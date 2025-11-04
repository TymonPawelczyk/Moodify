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
  <h1 class="">Display your Spotify profile data</h1>
</template>
<style scoped></style>
