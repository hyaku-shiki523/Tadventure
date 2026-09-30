<script setup>
import { onBeforeUnmount, ref } from 'vue'
import generalQuestions from '../data/general.json'
import entertainmentQuestions from '../data/entertainment.json'
import ChoiceButton from './ChoiceButton.vue'
import CountdownOverlay from './CountdownOverlay.vue'
import QuestionBoard from './QuestionBoard.vue'

const props = defineProps({
  questionSource: {
    type: String,
    default: 'questions'
  }
})

const answerMessage = ref('')
const selectedChoice = ref('')
const phase = ref('choice')
const questionIndex = ref(0)
let moveTimer
let judgmentTimer
let answerTimer
let nextQuestionTimer

let questionPool

if (props.questionSource === 'entertainment') {
  questionPool = entertainmentQuestions
} else {
  questionPool = generalQuestions
}

const gameQuestions = []

for (let difficulty = 1; difficulty <= 5; difficulty += 1) {
  const questionsAtDifficulty = questionPool.filter(
    (question) => question.difficulty === difficulty
  )
  const randomIndex = Math.floor(Math.random() * questionsAtDifficulty.length)

  gameQuestions.push(questionsAtDifficulty[randomIndex])
}

const currentQuestion = ref(gameQuestions[questionIndex.value])

function answer(choice) {
  if (phase.value !== 'choice') {
    return
  }

  selectedChoice.value = choice
  phase.value = 'moving'

  moveTimer = window.setTimeout(() => {
    phase.value = 'waiting'

    judgmentTimer = window.setTimeout(() => {
      phase.value = 'answer'

      answerTimer = window.setTimeout(() => {
        if (choice === currentQuestion.value.answer) {
          answerMessage.value = '正解'
        } else {
          answerMessage.value = '不正解'
        }

        if (questionIndex.value < 4) {
          nextQuestionTimer = window.setTimeout(() => {
            questionIndex.value += 1
            currentQuestion.value = gameQuestions[questionIndex.value]
            answerMessage.value = ''
            selectedChoice.value = ''
            phase.value = 'choice'
          }, 1500)
        }
      }, 1500)
    }, 3000)
  }, 200)
}

function answerByMouseButton(event) {
  if (event.button === 0) {
    answer('left')
  } else if (event.button === 2) {
    answer('right')
  }
}

onBeforeUnmount(() => {
  window.clearTimeout(moveTimer)
  window.clearTimeout(judgmentTimer)
  window.clearTimeout(answerTimer)
  window.clearTimeout(nextQuestionTimer)
})
</script>
<template>
  <section
    @mousedown.prevent="answerByMouseButton"
    @contextmenu.prevent
  >
    <CountdownOverlay
      :key="`countdown-${questionIndex}`"
      :start-delay="5900"
      :cancelled="selectedChoice !== ''"
    />

    <QuestionBoard
      :key="`question-${questionIndex}`"
      :text="currentQuestion.text"
    />

    <ChoiceButton
      :key="`left-${questionIndex}`"
      side="left"
      :label="currentQuestion.left"
      :selected="selectedChoice === 'left'"
      :dismissed="selectedChoice === 'right'"
      :clicked="selectedChoice === 'left'"
      :disabled="phase !== 'choice'"
      :hidden="phase === 'answer'"
      @choose="answer"
    />
    <ChoiceButton
      :key="`right-${questionIndex}`"
      side="right"
      :label="currentQuestion.right"
      :selected="selectedChoice === 'right'"
      :dismissed="selectedChoice === 'left'"
      :clicked="selectedChoice === 'right'"
      :disabled="phase !== 'choice'"
      :hidden="phase === 'answer'"
      @choose="answer"
    />

    <p
      v-if="answerMessage"
      class="answer-message"
    >
      {{ answerMessage }}
    </p>
  </section>
</template>

<style scoped>
.answer-message {
  position: fixed;
  top: 50%;
  left: 50%;
  z-index: 20;
  margin: 0;
  color: transparent;
  background: linear-gradient(180deg, #e60000 0%, #ff8a00 100%);
  background-clip: text;
  -webkit-background-clip: text;

  font-size: clamp(5rem, 14vw, 11rem);
  -webkit-text-stroke: 2px #ffffff;
    
  font-weight: 900;
  line-height: 1;
  transform: translate(-50%, -50%);
  animation: judgment-zoom-in 0.4s ease-out both;
}

@keyframes judgment-zoom-in {
  from {
    opacity: 0;
    transform: translate(-50%, -50%) scale(0.1);
  }

  to {
    opacity: 1;
    transform: translate(-50%, -50%) scale(1);
  }
}
</style>
