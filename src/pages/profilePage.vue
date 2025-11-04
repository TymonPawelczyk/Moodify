<script setup lang="ts">
import menuComponent from '@/components/menuComponent.vue'
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
  <menu-component />
  <h1 class="">Display your Spotify profile data</h1>
</template>
<style scoped></style>
