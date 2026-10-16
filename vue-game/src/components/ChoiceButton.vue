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
  disabled: {
    type: Boolean,
    default: false
  },
  selected: {
    type: Boolean,
    default: false
  },
  dismissed: {
    type: Boolean,
    default: false
  }
})

</script>

<template>
  <button
    :class="[`${side}-button`, { 'is-selected': selected, 'is-dismissed': dismissed }]"
    :disabled="disabled"
  >
    <span>{{ label }}</span>
  </button>
</template>

<style scoped>
.left-button,
.right-button {
  position: relative;
  display: flex;
  width: 100%;
  min-height: clamp(120px, 20vh, 200px);
  align-items: center;
  justify-content: center;
  padding: clamp(20px, 3vw, 36px);
  border: 1px solid var(--green-dim);
  background:
    linear-gradient(140deg, rgba(112, 255, 139, 0.07), transparent 54%),
    rgba(7, 15, 10, 0.96);
  box-shadow: inset 0 0 24px rgba(112, 255, 139, 0.025);
  color: var(--text);
  font-size: clamp(1.1rem, 2.8vw, 2.1rem);
  font-weight: 700;
  line-height: 1.5;
  cursor: pointer;
  overflow-wrap: anywhere;
  transition: border-color 160ms ease, background 160ms ease, box-shadow 160ms ease;
}

.left-button:disabled:not(.is-selected),
.right-button:disabled:not(.is-selected) {
  cursor: default;
  opacity: 0.58;
}

.left-button.is-selected,
.right-button.is-selected {
  border-color: var(--green);
  background: rgba(112, 255, 139, 0.14);
  box-shadow: inset 0 0 34px rgba(112, 255, 139, 0.12), 0 0 28px rgba(112, 255, 139, 0.16);
}

.is-dismissed {
  opacity: 0;
}

@media (max-width: 600px) {
  .left-button,
  .right-button {
    min-height: clamp(100px, 18vh, 150px);
  }
}

</style>
