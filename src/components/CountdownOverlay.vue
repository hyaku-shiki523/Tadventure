<script setup>
import { onBeforeUnmount, onMounted, ref, watch } from 'vue'

const props = defineProps({
  startDelay: {
    type: Number,
    default: 1500
  },
  cancelled: {
    type: Boolean,
    default: false
  }
})

const count = ref(null)
let startTimer
let countTimer

function stopCountdown() {
  window.clearTimeout(startTimer)
  window.clearInterval(countTimer)
  count.value = null
}

watch(() => props.cancelled, (cancelled) => {
  if (cancelled) {
    stopCountdown()
  }
})

onMounted(() => {
  startTimer = window.setTimeout(() => {
    count.value = 5

    countTimer = window.setInterval(() => {
      count.value -= 1

      if (count.value === 0) {
        window.clearInterval(countTimer)
      }
    }, 1000)
  }, props.startDelay)
})

onBeforeUnmount(() => {
  stopCountdown()
})
</script>

<template>
  <Transition name="countdown">
    <div
      v-if="count !== null"
      class="countdown-overlay"
      aria-live="assertive"
      aria-label="カウントダウン"
    >
      <span :key="count" class="countdown-number">{{ count }}</span>
    </div>
  </Transition>
</template>

<style scoped>
.countdown-overlay {
  position: fixed;
  inset: 0;
  z-index: 10;
  display: grid;
  place-items: center;
  pointer-events: none;
}

.countdown-number {
  color: transparent;
  background: linear-gradient(180deg, #e60000 0%, #ff8a00 100%);
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-stroke: 3px #ffffff;
  font-size: 400px;
  font-weight: bold;
  line-height: 1;
  text-shadow: 0 8px 24px rgba(0, 0, 0, 0.35);
  animation: countdown-pop 1s ease both;
}

.countdown-enter-active,
.countdown-leave-active {
  transition: opacity 0.25s ease;
}

.countdown-enter-from,
.countdown-leave-to {
  opacity: 0;
}

@keyframes countdown-pop {
  0% {
    opacity: 0;
    transform: scale(1.45);
  }

  25% {
    opacity: 1;
    transform: scale(1);
  }

  80% {
    opacity: 1;
    transform: scale(1);
  }

  100% {
    opacity: 0;
    transform: scale(0.8);
  }
}
</style>
