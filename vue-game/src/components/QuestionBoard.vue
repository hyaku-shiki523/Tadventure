<script setup>
defineProps({
  text: {
    type: String,
    required: true
  }
})
</script>

<template>
  <div class="question-frame">
    <div class="question-board">
      <h2>{{ text }}</h2>
    </div>
  </div>
</template>

<style scoped>
.question-frame {
  position: relative;
  width: 100%;
  align-self: stretch;
  animation: question-board-zoom 0.2s both;
}

.question-board {
  position: relative;
  display: flex;
  min-height: clamp(130px, 20vh, 210px);
  flex-direction: column;
  justify-content: center;
  gap: 18px;
  padding: clamp(20px, 3vw, 36px);
  border: 1px solid var(--green-dim);
  background:
    linear-gradient(110deg, rgba(112, 255, 139, 0.07), transparent 38%),
    rgba(5, 12, 8, 0.94);
  box-shadow: inset 0 0 40px rgba(112, 255, 139, 0.035), 0 18px 55px rgba(0, 0, 0, 0.3);
  text-align: center;
}

.question-board::before,
.question-board::after {
  position: absolute;
  width: 18px;
  height: 18px;
  border-color: var(--green);
  border-style: solid;
  content: '';
}

.question-board::before {
  top: -1px;
  left: -1px;
  border-width: 2px 0 0 2px;
}

.question-board::after {
  right: -1px;
  bottom: -1px;
  border-width: 0 2px 2px 0;
}

.question-board h2 {
  position: relative;
  z-index: 1;
  margin: 0;
  color: var(--text);
  font-size: clamp(1.35rem, 2.8vw, 2.2rem);
  max-width: 32ch;
  align-self: center;
  font-weight: 800;
  letter-spacing: 0.01em;
  line-height: 1.45;
  animation: question-content-show 0.2s 0.15s both;
}

@keyframes question-board-zoom {
  from {
    opacity: 0;
    transform: translateY(12px) scale(0.98);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

@keyframes question-content-show {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
</style>
