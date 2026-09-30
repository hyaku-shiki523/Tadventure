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
    <div class="chain-up"></div>
    <div class="question-board">
      <h2>{{ text }}</h2>
    </div>
    <div class="chain-down"></div>
  </div>
</template>

<style scoped>
.question-frame {
  position: relative;
  transform: translateY(-60%);
}

.question-board {
  position: relative;
  width: min(2100px);
  min-height: 240px;
  background-image: url('/woodboard2.png');
  background-size: cover;
  background-position: center;
  transform-origin: center center;
  z-index: 1;
  animation: question-board-zoom 0.2s 1.3s both;
}

.question-board::before,
.question-board::after {
  position: absolute;
  top: 48%;
  color: white;
  font-size: 80px;
  font-weight: 900;
  line-height: 1;
  text-shadow:
    3px 0 3px #000000,
    -3px 0 3px #000000,
    0 3px 3px #000000,
    0 -3px 3px #000000;
  transform: translateY(-50%);
  opacity: 0;
  animation: question-content-show 0s 2.4s forwards;
}

.question-board::before {
  left: 140px;
  content: '◀';
  z-index: 5;
}

.question-board::after {
  right: 140px;
  content: '▶';
  z-index: 5;
}

.question-board h2 {
  position: relative;
  z-index: 10;
  font-size: 100px;
  font-family: "Zen Antique", "Yuji Boku", serif;
  font-weight: 900;
  text-shadow:
    3px 0 3px #000000,
    -3px 0 3px #000000,
    0 3px 3px #000000,
    0 -3px 3px #000000;
  transform: translateY(50px);
  opacity: 0;
  animation: question-content-show 0s 2.4s forwards;
}

.chain-up,
.chain-down {
  position: absolute;
  left: 50%;
  width: 130vw;
  height: 90px;
  background-position: center;
  background-size: 100% auto;
  background-repeat: no-repeat;
  transform: translateX(-50%);
  z-index: 3;
  animation-duration: 0.8s;
  animation-timing-function: ease-out;
}

.chain-up {
  bottom: 58%;
  background-image: url('/chainup.png');
  animation-name: chain-up-slide;
}

.chain-down {
  top: 54%;
  background-image: url('/chaindown.png');
  animation-name: chain-down-slide;
}

@keyframes question-board-zoom {
  from {
    opacity: 0;
    transform: scale(0.1);
  }
  to {
    opacity: 1;
    transform: scale(1);
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

@keyframes chain-up-slide {
  from {
    transform: translateX(100vw);
  }
  to {
    transform: translateX(-50%);
  }
}

@keyframes chain-down-slide {
  from {
    transform: translateX(-230vw);
  }
  to {
    transform: translateX(-50%);
  }
}
</style>
