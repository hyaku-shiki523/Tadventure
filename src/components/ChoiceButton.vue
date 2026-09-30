<script setup>
const props = defineProps({
  side: {
    type: String,
    required: true
  },
  label: {
    type: String,
    required: true
  },
  clicked: {
    type: Boolean,
    default: false
  },
  dismissed: {
    type: Boolean,
    default: false
  },
  disabled: {
    type: Boolean,
    default: false
  },
  hidden: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['choose'])

function choose() {
  if (props.disabled || props.hidden) {
    return
  }

  emit('choose', props.side)
}
</script>

<template>
  <button
    :class="[
      `${side}-button`,
      {
        'left-clicked': clicked && side === 'left',
        'right-clicked': clicked && side === 'right',
        'is-dismissed': dismissed,
        'is-hidden': hidden
      }
    ]"
    :disabled="disabled"
    @click="choose"
  >
    {{ label }}
  </button>
</template>

<style scoped>
.left-button,
.right-button {
  display: inline-flex;
  vertical-align: top;
  width: 650px;
  height: 650px;
  align-items: center;
  justify-content: center;
  margin: 0 130px;
  border: 23px solid transparent;
  border-radius: 0;
  background:
    linear-gradient(#ffffff, #ffffff) padding-box,
    linear-gradient(
      135deg,
      #30363d 0%,
      #6e747a 15%,
      #9c9fa3 25%,
      #6e747a 35%,
      #30363d 50%,
      #6e747a 65%,
      #9c9fa3 75%,
      #6e747a 85%,
      #454c54 100%
    ) border-box;
  box-shadow:
    0 18px 18px rgba(0, 0, 0, 0.35),
    inset 0 0 0 2px rgba(255, 255, 255, 0.35);
  color: #000000;
  font-family: Arial, sans-serif;
  font-size: 90px;
  font-weight: 900;
  cursor: pointer;
  transform: translateY(-10%);
  animation-duration: 0.5s;
  animation-timing-function: ease-out;
  animation-delay: 3.9s;
  animation-fill-mode: both;
  animation-iteration-count: 1;
}

.left-button {
  transform-origin: right;
  animation-name: left-button-zoom;
}

.right-button {
  transform-origin: left;
  animation-name: right-button-zoom;
}

.right-clicked {
  animation: right-slide-center 0.2s linear forwards;
}

.left-button.is-dismissed {
  animation: left-slide-out 0.4s linear forwards;
}

.left-clicked {
  animation: left-slide-center 0.2s linear forwards;
}

.right-button.is-dismissed {
  animation: right-slide-out 0.4s linear forwards;
}

.is-hidden {
  visibility: hidden;
}

@keyframes left-button-zoom {
  from {
    opacity: 0;
    transform: translateY(-10%) scale(0.1);
  }
  to {
    opacity: 1;
    transform: translateY(-10%) scale(1);
  }
}

@keyframes right-button-zoom {
  from {
    opacity: 0;
    transform: translateY(-10%) scale(0.1);
  }
  to {
    opacity: 1;
    transform: translateY(-10%) scale(1);
  }
}

@keyframes right-slide-center {
  to {
    transform: translate(-70%, -10%);
  }
}

@keyframes left-slide-out {
  to {
    opacity: 0;
    transform: translateX(-100vw) translateY(-10%);
  }
}

@keyframes left-slide-center {
  to {
    transform: translate(70%, -10%);
  }
}

@keyframes right-slide-out {
  to {
    opacity: 0;
    transform: translateX(100vw) translateY(-10%);
  }
}

</style>
