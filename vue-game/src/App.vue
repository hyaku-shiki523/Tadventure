<script setup>
import { onBeforeUnmount, onMounted, ref } from 'vue'
import vivantTheme from './assets/vivant-theme.mp3'
import TitleScreen from './components/TitleScreen.vue'
import IntroScreen from './components/IntroScreen.vue'
import QuestionScreen from './components/QuestionScreen.vue'

const screen = ref('title')
const backgroundVideo = ref(null)
const themeAudio = ref(null)
let themeStarted = false

function startIntro() {
  screen.value = 'intro'
}

function handleFirstRightArrow(event) {
  if (event.key !== 'ArrowRight' || screen.value !== 'intro' || themeStarted) {
    return
  }

  playTheme()
}

function playTheme() {
  if (themeStarted) {
    return
  }

  themeStarted = true
  themeAudio.value?.play().catch((error) => {
    console.error('VIVANTテーマ音源を再生できませんでした:', error)
  })
}

onMounted(() => window.addEventListener('keydown', handleFirstRightArrow))
onBeforeUnmount(() => window.removeEventListener('keydown', handleFirstRightArrow))

function startGame() {
  screen.value = 'question'
  backgroundVideo.value?.play().catch((error) => {
    console.error('背景動画を再生できませんでした:', error)
  })
}

function skipIntro() {
  playTheme()
  startGame()
}

</script>

<template>
  <main>
    <video
      ref="backgroundVideo"
      class="background-video"
      :class="{ 'is-playing': screen === 'question' }"
      src="/tunnel%20straight.mp4"
      muted
      loop
      playsinline
      preload="auto"
      aria-hidden="true"
    />
    <audio ref="themeAudio" :src="vivantTheme" preload="auto" />

    <Transition name="screen" mode="out-in">
      <TitleScreen
        v-if="screen === 'title'"
        key="title"
        @start="startIntro"
      />
      <IntroScreen
        v-else-if="screen === 'intro'"
        key="intro"
        @complete="startGame"
        @skip="skipIntro"
      />
      <QuestionScreen
        v-else
        key="question"
      />
    </Transition>
  </main>
</template>

<style scoped>
main {
  position: relative;
  isolation: isolate;
  min-height: 100vh;
  min-height: 100svh;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  overflow-x: hidden;
  overflow-y: auto;
  padding: clamp(18px, 4vw, 56px);
  background:
    radial-gradient(ellipse at 50% 42%, rgba(22, 74, 38, 0.28), transparent 54%),
    #050907;
}

.background-video {
  position: fixed;
  inset: 0;
  z-index: -2;
  width: 100%;
  height: 100%;
  object-fit: cover;
  opacity: 0.12;
  filter: saturate(0.45) hue-rotate(65deg) brightness(0.62);
  pointer-events: none;
  transition: opacity 0.2s ease;
}

.background-video.is-playing {
  opacity: 0.22;
}

main::before {
  position: fixed;
  inset: 0;
  z-index: -1;
  background-image:
    linear-gradient(rgba(112, 255, 139, 0.035) 1px, transparent 1px),
    linear-gradient(90deg, rgba(112, 255, 139, 0.035) 1px, transparent 1px);
  background-size: 48px 48px;
  content: '';
  pointer-events: none;
}

.screen-enter-active,
.screen-leave-active {
  transition: opacity 220ms ease, transform 220ms ease;
}

.screen-enter-from {
  opacity: 0;
  transform: translateY(10px);
}

.screen-leave-to {
  opacity: 0;
  transform: translateY(-8px);
}
</style>