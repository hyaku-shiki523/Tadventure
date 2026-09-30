<script setup>
import { ref } from 'vue'
import TitleScreen from './components/TitleScreen.vue'
import QuestionScreen from './components/QuestionScreen.vue'

const screen = ref('title')
const questionSource = ref('questions')
const backgroundVideo = ref(null)

function startGame(source) {
  questionSource.value = source
  screen.value = 'question'
  backgroundVideo.value?.play().catch((error) => {
    console.error('背景動画を再生できませんでした:', error)
  })
}
</script>

<template>
  <main :class="{ 'game-active': screen === 'question' }">
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

    <TitleScreen
      v-if="screen === 'title'"
      @start="startGame"
    />

    <QuestionScreen
      v-else-if="screen === 'question'"
      :question-source="questionSource"
    />
  </main>
</template>

<style scoped>
main {
  position: relative;
  isolation: isolate;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  box-sizing: border-box;
  padding: 20px;
}

:global(main.game-active),
:global(main.game-active *) {
  cursor: none !important;
}

.background-video {
  position: fixed;
  inset: 0;
  z-index: -1;
  width: 100%;
  height: 100%;
  object-fit: cover;
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.2s ease;
}

.background-video.is-playing {
  opacity: 1;
}
</style>