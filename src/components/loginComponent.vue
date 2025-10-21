<script setup lang="ts">
import { ref, onMounted } from 'vue'

const generateRandomString = (length: number): string => {
  const possible = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789'
  const values = globalThis.crypto.getRandomValues(new Uint8Array(length))
  return Array.from(values)
    .map((x) => possible[x % possible.length])
    .join('')
}

const sha256 = async (plain: string): Promise<ArrayBuffer> => {
  const encoder = new TextEncoder()
  const data = encoder.encode(plain)
  return globalThis.crypto.subtle.digest('SHA-256', data)
}

const base64encode = (input: ArrayBuffer): string => {
  return btoa(String.fromCharCode(...new Uint8Array(input)))
    .replace(/=/g, '')
    .replace(/\+/g, '-')
    .replace(/\//g, '_')
}

const clientId = import.meta.env.VITE_CLIENT_ID
const redirectUri = import.meta.env.VITE_REDIRECT_URI

const scope = 'user-read-private user-read-email'
const authBase = 'https://accounts.spotify.com/authorize'
const tokenUrl = 'https://accounts.spotify.com/api/token'

const codeVerifier = ref('')
const codeChallenge = ref('')
const state = ref('')

const login = async (): Promise<void> => {
  // generate PKCE values
  codeVerifier.value = generateRandomString(64)
  const hashed = await sha256(codeVerifier.value)
  codeChallenge.value = base64encode(hashed)
  state.value = generateRandomString(16)

  // store for exchange step
  window.localStorage.setItem('code_verifier', codeVerifier.value)
  window.localStorage.setItem('pkce_state', state.value)

  // build authorize URL & redirect
  const authUrl = new URL(authBase)
  const params = {
    response_type: 'code',
    client_id: clientId,
    scope,
    code_challenge_method: 'S256',
    code_challenge: codeChallenge.value,
    redirect_uri: redirectUri,
    state: state.value,
  }
  authUrl.search = new URLSearchParams(params).toString()
  window.location.href = authUrl.toString()
}

const getToken = async (code: string) => {
  const storedVerifier = window.localStorage.getItem('code_verifier')
  if (!storedVerifier) {
    console.error('No code_verifier found in localStorage.')
    return null
  }

  const res = await fetch(tokenUrl, {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      client_id: clientId,
      grant_type: 'authorization_code',
      code,
      redirect_uri: redirectUri,
      code_verifier: storedVerifier,
    }),
  })

  if (!res.ok) {
    const err = await res.json().catch(() => ({ error: 'unknown' }))
    console.error('Token exchange failed:', err)
    return null
  }

  const data = await res.json()
  window.localStorage.setItem('access_token', data.access_token)
  if (data.refresh_token) window.localStorage.setItem('refresh_token', data.refresh_token)
  return data
}

onMounted(async () => {
  const params = new URLSearchParams(window.location.search)
  const code = params.get('code')
  const returnedState = params.get('state')
  const storedState = window.localStorage.getItem('pkce_state')

  if (code) {
    if (storedState && returnedState !== storedState) {
      console.error('State mismatch. Possible CSRF attack.')
      return
    }
    await getToken(code)
    // Clean up URL and stored values
    const url = new URL(window.location.href)
    url.search = ''
    window.history.replaceState({}, document.title, url.toString())
    window.localStorage.removeItem('pkce_state')
    window.localStorage.removeItem('code_verifier')
  }
})
</script>

<template>
  <button
    @click="login"
    class="md:flex justify-center items-center bg-green-500 rounded-4xl p-2 hover:shadow-lg transition-shadow duration-200"
  >
    <img
      src="./../assets/2024-spotify-logo-icon/Primary_Logo_Black_RGB.svg"
      alt="Spotify logo"
      class="h-8"
    />
    <p class="m-1 font-semibold">Login with Spotify</p>
  </button>
</template>

<style scoped></style>
