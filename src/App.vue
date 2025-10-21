<script setup lang="ts">
import HomePage from './pages/homePage.vue'
import AboutPage from './pages/aboutPage.vue'
import ProfilePage from './pages/profilePage.vue'

import { ref, computed, onMounted, defineComponent, h } from 'vue'
import type { Component } from 'vue'

const NotFound = defineComponent({
  name: 'NotFound',
  setup() {
    return () => h('div', 'Not Found')
  },
})

const routes: Record<string, Component> = {
  '/': HomePage,
  '/about': AboutPage,
  '/profile': ProfilePage,
}

const currentPath = ref(window.location.hash)

const currentView = computed<Component>(() => routes[currentPath.value.slice(1) || '/'] || NotFound)

onMounted(() => {
  window.addEventListener('hashchange', () => {
    currentPath.value = window.location.hash
  })
})
</script>

<template>
  <router-view />
  <component :is="currentView" />
</template>

<style scoped></style>
