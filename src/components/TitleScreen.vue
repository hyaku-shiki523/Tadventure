<script setup>
import { ref } from 'vue'

const emit = defineEmits(['start'])
const selectedQuestionSet = ref('questions')

function startGame() {
  emit('start', selectedQuestionSet.value)
}
</script>

<template>
  <section>
    <h1>トロッコアドベンチャー</h1>
    <p>出題ジャンル</p>
    <div class="question-set-options">
      <button
        :aria-pressed="selectedQuestionSet === 'questions'"
        @click="selectedQuestionSet = 'questions'"
      >
        general
      </button>
      <button
        :aria-pressed="selectedQuestionSet === 'entertainment'"
        @click="selectedQuestionSet = 'entertainment'"
      >
        entertainment
      </button>
    </div>
    <button @click="startGame">スタート</button>
  </section>
</template>

<style scoped>
.question-set-options {
  display: flex;
  justify-content: center;
  gap: 12px;
  margin: 16px 0;
}

.question-set-options button[aria-pressed='true'] {
  outline: 3px solid #aa3bff;
  outline-offset: 3px;
}

</style>
