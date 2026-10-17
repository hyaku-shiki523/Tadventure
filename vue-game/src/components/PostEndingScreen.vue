<script setup>
import { onBeforeUnmount, onMounted, ref } from 'vue'

const messages = [
  '本日はありがとうございました。',
  'VIVANTを観ている方も、そうでない方も楽しんでいただけたのなら、幸いです。',
  'それでは、トロッコ前方左側からお帰りください。'
]
const visibleMessage = ref('')
let typingTimer
let advanceTimer
const activeVoices = new Set()
const characterDelay = 45

function playVoiceSound() {
  const audio = new Audio('/voice_sans.mp3')
  activeVoices.add(audio)
  audio.addEventListener('ended', () => activeVoices.delete(audio), { once: true })
  audio.play().catch((error) => {
    activeVoices.delete(audio)
    console.error('V-Iの音声を再生できませんでした:', error)
  })
}

function showMessage(index) {
  const characters = Array.from(messages[index])
  let characterIndex = 0
  visibleMessage.value = ''

  typingTimer = window.setInterval(() => {
    visibleMessage.value += characters[characterIndex]
    playVoiceSound()
    characterIndex += 1

    if (characterIndex === characters.length) {
      window.clearInterval(typingTimer)
      if (index < messages.length - 1) {
        advanceTimer = window.setTimeout(() => showMessage(index + 1), 1500)
      }
    }
  }, characterDelay)
}

onMounted(() => showMessage(0))

onBeforeUnmount(() => {
  window.clearInterval(typingTimer)
  window.clearTimeout(advanceTimer)
  for (const audio of activeVoices) {
    audio.pause()
    audio.removeAttribute('src')
  }
  activeVoices.clear()
})
</script>

<template>
  <section class="post-ending-panel" aria-live="polite">
    <h1>V-I</h1>
    <p>{{ visibleMessage }}<span class="typing-cursor">▋</span></p>
  </section>
</template>

<style scoped>
.post-ending-panel {
  display: flex;
  align-self: center;
  width: min(100%, 900px);
  min-height: min(460px, 68vh);
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: clamp(28px, 6vw, 64px);
  border: 1px solid var(--green-dim);
  background:
    linear-gradient(110deg, rgba(112, 255, 139, 0.06), transparent 42%),
    rgba(1, 5, 2, 0.97);
  box-shadow: 0 20px 70px rgba(0, 0, 0, 0.48), inset 0 0 42px rgba(112, 255, 139, 0.035);
  text-align: center;
}

h1 {
  margin: 0 0 24px;
  color: var(--green-bright);
  font-family: var(--mono);
  font-size: clamp(1rem, 2vw, 1.25rem);
  letter-spacing: 0.1em;
}

p {
  display: grid;
  min-height: 100px;
  place-items: center;
  margin: 0;
  color: var(--text);
  font-family: var(--mono);
  font-size: clamp(1rem, 2.4vw, 1.45rem);
  line-height: 2;
}

.typing-cursor {
  display: inline-block;
  margin-left: 3px;
  color: var(--green);
  animation: blink 0.8s steps(2, start) infinite;
}

@keyframes blink {
  to {
    visibility: hidden;
  }
}
</style>
