<script setup>
import { onBeforeUnmount, onMounted, ref } from 'vue'

const emit = defineEmits(['start'])
const started = ref(false)
const bgmEnabled = ref(false)
const bgmCheckbox = ref(null)

function handleKeydown(event) {
  if (started.value) {
    return
  }

  if (event.code === 'Space' && event.target !== bgmCheckbox.value) {
    event.preventDefault()
    if (!event.repeat) {
      bgmEnabled.value = !bgmEnabled.value
    }
  } else if (event.key === 'Enter') {
    event.preventDefault()
    started.value = true
    emit('start', bgmEnabled.value)
  }
}

onMounted(() => window.addEventListener('keydown', handleKeydown))
onBeforeUnmount(() => window.removeEventListener('keydown', handleKeydown))
</script>

<template>
  <section class="title-panel">
    <h1>トロッコ<span>アドベンチャー</span></h1>
    <label class="bgm-toggle">
      <input ref="bgmCheckbox" v-model="bgmEnabled" type="checkbox">
      <span class="checkbox-mark" aria-hidden="true"></span>
      <span>BGMあり</span>
    </label>
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

.bgm-toggle {
  position: relative;
  display: inline-flex;
  align-items: center;
  gap: 10px;
  margin-top: 24px;
  color: var(--text);
  font-size: 1rem;
  cursor: pointer;
}

.bgm-toggle input {
  position: absolute;
  top: 0;
  left: 0;
  width: 1px;
  height: 1px;
  opacity: 0;
}

.checkbox-mark {
  display: grid;
  width: 22px;
  height: 22px;
  place-items: center;
  border: 1px solid var(--green-dim);
  background: rgba(3, 9, 5, 0.9);
}

.bgm-toggle input:checked + .checkbox-mark {
  border-color: var(--green);
  background: rgba(112, 255, 139, 0.18);
  box-shadow: 0 0 12px rgba(112, 255, 139, 0.22);
}

.bgm-toggle input:checked + .checkbox-mark::after {
  width: 10px;
  height: 6px;
  border-bottom: 2px solid var(--green-bright);
  border-left: 2px solid var(--green-bright);
  content: '';
  transform: translateY(-2px) rotate(-45deg);
}

.bgm-toggle input:focus-visible + .checkbox-mark {
  outline: 2px solid var(--green-bright);
  outline-offset: 3px;
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
