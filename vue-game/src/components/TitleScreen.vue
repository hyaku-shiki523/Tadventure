<script setup>
import { onBeforeUnmount, onMounted, ref } from 'vue'

const emit = defineEmits(['start'])
const started = ref(false)

function handleKeydown(event) {
  if (event.key === 'Enter' && !started.value) {
    event.preventDefault()
    started.value = true
    emit('start')
  }
}

onMounted(() => window.addEventListener('keydown', handleKeydown))
onBeforeUnmount(() => window.removeEventListener('keydown', handleKeydown))
</script>

<template>
  <section class="title-panel">
    <h1>トロッコ<span>アドベンチャー</span></h1>
    <div class="start-prompt">
      <span class="start-indicator">▶</span>
      <span>blue@walkerとの通信を開始</span>
    </div>
  </section>
</template>

<style scoped>
.title-panel {
  position: relative;
  width: min(100%, 760px);
  padding: clamp(28px, 6vw, 64px);
  border: 1px solid var(--line);
  background:
    linear-gradient(135deg, rgba(112, 255, 139, 0.06), transparent 38%),
    var(--panel);
  box-shadow: 0 24px 90px rgba(0, 0, 0, 0.42), 0 0 50px rgba(50, 220, 94, 0.06);
  text-align: center;
}

.title-panel::before,
.title-panel::after {
  position: absolute;
  width: 18px;
  height: 18px;
  border-color: var(--green);
  border-style: solid;
  content: '';
  pointer-events: none;
}

.title-panel::before {
  top: -1px;
  left: -1px;
  border-width: 2px 0 0 2px;
}

.title-panel::after {
  right: -1px;
  bottom: -1px;
  border-width: 0 2px 2px 0;
}

h1 {
  margin: 0;
  color: #f1fff3;
  font-size: clamp(2rem, 6.3vw, 4rem);
  font-weight: 800;
  letter-spacing: 0.02em;
  line-height: 1.25;
}

h1 span {
  display: block;
  color: var(--green);
}

.start-prompt {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 12px;
  margin-top: 28px;
  color: var(--green-bright);
  font-family: var(--mono);
  font-size: clamp(0.85rem, 2.6vw, 1.1rem);
  letter-spacing: 0.06em;
  animation: prompt-pulse 1.2s ease-in-out infinite alternate;
}

.start-indicator {
  color: var(--green);
}

@keyframes prompt-pulse {
  to {
    opacity: 0.55;
  }
}
</style>
