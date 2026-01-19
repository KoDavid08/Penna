<script setup lang="ts">
import { ref, computed } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()
const radialOpen = ref(false)
const activeIndex = ref<number | null>(null)
const menuOpen = ref(false)
const darkMode = ref(false)
const selectedLang = ref('HU')

const radialItems = [
  { label: 'Főoldal', to: '/' },
  { label: 'Jegyek', to: '/grades' },
  { label: 'Üzenetek', to: '/messages' }
]

let centerX = 0
let centerY = 0

const onPointerDown = (e: PointerEvent) => {
  const rect = (e.currentTarget as HTMLElement).getBoundingClientRect()
  centerX = rect.left + rect.width / 2
  centerY = rect.top + rect.height / 2
  radialOpen.value = true
  activeIndex.value = null
  ;(e.currentTarget as HTMLElement).setPointerCapture(e.pointerId)
}

const onPointerMove = (e: PointerEvent) => {
  if (!radialOpen.value) return
  const dx = e.clientX - centerX
  const dy = e.clientY - centerY
  const distance = Math.sqrt(dx * dx + dy * dy)
  if (distance < 40) {
    activeIndex.value = null
    return
  }
  let angle = (Math.atan2(dy, dx) * 180) / Math.PI
  if (angle <= 0 && angle >= -180) {
    if (angle < -120) activeIndex.value = 0
    else if (angle < -60) activeIndex.value = 1
    else activeIndex.value = 2
  } else activeIndex.value = null
}

const onPointerUp = () => {
  if (activeIndex.value !== null) router.push(radialItems[activeIndex.value].to)
  radialOpen.value = false
  activeIndex.value = null
}

const toggleDarkMode = () => {
  darkMode.value = !darkMode.value
  if (darkMode.value) document.documentElement.classList.add('dark')
  else document.documentElement.classList.remove('dark')
}

const toggleMenu = () => menuOpen.value = !menuOpen.value
</script>

<template>
  <header class="navbar">
    <div class="logo">MyApp</div>
    <nav :class="{ open: menuOpen }">
      <ul>
        <li @click="router.push('/')">Főoldal</li>
        <li @click="router.push('/grades')">Jegyek</li>
        <li @click="router.push('/messages')">Üzenetek</li>
      </ul>
      <div class="nav-actions">
        <button @click="toggleDarkMode">{{ darkMode ? 'Világos' : 'Sötét' }}</button>
        <select v-model="selectedLang">
          <option value="HU">HU</option>
          <option value="EN">EN</option>
        </select>
        <button @click="router.push('/login')">Bejelentkezés</button>
      </div>
    </nav>
    <div class="hamburger" @click="toggleMenu">
      <span></span><span></span><span></span>
    </div>
  </header>

  <div class="layout-wrapper">
    <div v-if="radialOpen" class="radial-blur-overlay"></div>

    <div v-if="radialOpen" class="radial-menu-system">
      <div
        v-for="(item, i) in radialItems"
        :key="i"
        class="sector-wrapper"
        :class="{ active: activeIndex === i }"
        :style="{ transform: `rotate(${-60 + (i * 60)}deg)` }"
      >
        <div class="label-anchor">
          <span class="label-text">{{ item.label }}</span>
        </div>
      </div>
    </div>

    <div
      class="trigger-button"
      @pointerdown="onPointerDown"
      @pointermove="onPointerMove"
      @pointerup="onPointerUp"
      @pointercancel="onPointerUp"
    >
      <div class="icon" :class="{ rotated: radialOpen }">+</div>
    </div>
  </div>

  <div class="router-view-container">
    <RouterView />
  </div>
</template>

<style scoped>
:root {
  --navbar-height: 60px;
}

body {
  margin: 0;
  font-family: 'Arial', sans-serif;
}

body.dark {
  background: #121212;
  color: #fff;
}

.navbar {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: var(--navbar-height);
  background: #fff;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 20px;
  z-index: 3000;
  box-shadow: 0 2px 8px rgba(0,0,0,0.15);
}

body.dark .navbar {
  background: #1e1e1e;
}

.navbar .logo {
  font-weight: bold;
  font-size: 18px;
}

.navbar nav {
  display: flex;
  align-items: center;
  gap: 20px;
}

.navbar nav ul {
  display: flex;
  list-style: none;
  margin: 0;
  padding: 0;
  gap: 20px;
}

.navbar nav ul li {
  cursor: pointer;
}

.nav-actions {
  display: flex;
  align-items: center;
  gap: 10px;
}

.navbar .hamburger {
  display: none;
  flex-direction: column;
  gap: 5px;
  cursor: pointer;
}

.navbar .hamburger span {
  width: 25px;
  height: 3px;
  background: #000;
}

body.dark .hamburger span {
  background: #fff;
}

.navbar nav.open {
  display: flex;
  flex-direction: column;
  position: absolute;
  top: var(--navbar-height);
  right: 0;
  background: inherit;
  width: 200px;
  padding: 10px;
}

.layout-wrapper {
  margin-top: var(--navbar-height);
  position: relative;
}

.router-view-container {
  position: relative;
  z-index: 1;
  width: 100vw;
  height: 100svh;
  overflow-y: auto;
  padding-bottom: 100px;
}

.trigger-button {
  position: fixed;
  bottom: 60px;
  left: 50%;
  transform: translateX(-50%);
  width: 64px;
  height: 64px;
  background: #000;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2000;
  touch-action: none;
  box-shadow: 0 8px 20px rgba(0,0,0,0.3);
}

.icon {
  color: white;
  font-size: 32px;
  transition: transform 0.2s ease;
}

.icon.rotated {
  transform: rotate(45deg);
}

.radial-blur-overlay {
  position: fixed;
  inset: 0;
  background: rgba(255, 255, 255, 0.3);
  backdrop-filter: blur(12px);
  z-index: 900;
}

.radial-menu-system {
  position: fixed;
  bottom: calc(60px + 32px);
  left: 50%;
  width: 0;
  height: 0;
  z-index: 1000;
}

.sector-wrapper {
  position: absolute;
  bottom: 0;
  left: -100px;
  width: 200px;
  height: 200px;
  transform-origin: bottom center;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  opacity: 0.3;
  transition: opacity 0.2s, width 0.2s, height 0.2s;
  clip-path: polygon(50% 100%, 0 0, 100% 0);
}

.sector-wrapper.active {
  opacity: 1;
  background: radial-gradient(circle at bottom center, rgba(0,0,0,0.1) 0%, transparent 80%);
}

.label-anchor {
  margin-top: 20px;
  transition: transform 0.2s;
}

.sector-wrapper.active .label-anchor {
  transform: scale(1.15) !important;
}

.label-text {
  background: #000;
  color: #fff;
  padding: 10px 20px;
  border-radius: 50px;
  font-weight: 600;
  font-size: 15px;
  white-space: nowrap;
  box-shadow: 0 4px 15px rgba(0,0,0,0.2);
}

/* Responsive */
@media (max-width: 768px) {
  .navbar nav {
    display: none;
  }
  .navbar .hamburger {
    display: flex;
  }
  .sector-wrapper {
    width: 200px;
    height: 200px;
    left: -100px;
  }
}

@media (max-width: 480px) {
  .sector-wrapper {
    width: 160px;
    height: 160px;
    left: -80px;
  }
}
</style>
