<script setup>
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'
import mixedQuestions from '../data/mixed.json'
import ChoiceButton from './ChoiceButton.vue'
import QuestionBoard from './QuestionBoard.vue'

const questions = Array.from({ length: 5 }, (_, index) => {
  const difficulty = index + 1
  const candidates = mixedQuestions.filter((question) => question.difficulty === difficulty)
  const question = candidates[Math.floor(Math.random() * candidates.length)]

  if (!question) {
    throw new Error(`難易度${difficulty}の問題が見つかりません。`)
  }

  return question
})

const questionIndex = ref(0)
const hp = ref(100)
const wrongAnswerCount = ref(0)
const pendingChoice = ref(null)
const selectionPending = ref(false)
const animationStarted = ref(false)
const resolved = ref(false)
const interfaceVisible = ref(false)
const gameFinished = ref(false)
const resultVisible = ref(false)
const messagesVisible = ref(false)
const countdownValue = ref(null)
const blueMessage = ref('')
const enemyMessage = ref('リバイバル社の幹部が標的を確認。')
const result = ref(null)
const pacing = {
  countdownLead: 2500,
  countdownStep: 1500,
  messageCharacter: 42,
  choiceAnimation: 2200,
  resultReveal: 2200,
  resultMessageDelay: 2200,
  nextQuestionDelay: 4000,
  interfaceIntro: 2500
}
const currentQuestion = computed(() => questions[questionIndex.value])
const movingClass = computed(() => {
  if (!animationStarted.value || !pendingChoice.value) {
    return ''
  }

  return pendingChoice.value === 'left' ? 'move-right' : 'move-left'
})

let introTimer
let blueTypingTimer
let enemyTypingTimer
let advanceTimer
let resultTimer
let messageDelayTimer
let selectionTimer
let bombTimer
let countdownStartTimer
let countdownTimer
const activeAudioPlayers = new Set()
const activeAudioSources = new Set()
const audioBuffers = new Map()
const soundEffects = [
  '/undertale-ding.mp3',
  '/undertale-bomb-explosion.mp3',
  '/undertale-damage-taken.mp3',
  '/カーソル移動5.mp3'
]
let audioContext

function startCountdown() {
  window.clearTimeout(countdownStartTimer)
  window.clearInterval(countdownTimer)
  countdownValue.value = null
  countdownStartTimer = window.setTimeout(() => {
    countdownValue.value = 5
    countdownTimer = window.setInterval(() => {
      if (countdownValue.value === 0) {
        countdownValue.value = null
        window.clearInterval(countdownTimer)
        return
      }

      countdownValue.value -= 1
    }, pacing.countdownStep)
  }, pacing.countdownLead)
}

function loadAudioBuffer(source) {
  if (!audioContext) {
    return Promise.reject(new Error('Web Audio APIが初期化されていません。'))
  }

  if (!audioBuffers.has(source)) {
    const bufferPromise = fetch(source)
      .then((response) => {
        if (!response.ok) {
          throw new Error(`音声ファイルの取得に失敗しました (${response.status}): ${source}`)
        }

        return response.arrayBuffer()
      })
      .then((buffer) => audioContext.decodeAudioData(buffer))
    audioBuffers.set(source, bufferPromise)
  }

  return audioBuffers.get(source)
}

function unlockAudio() {
  if (audioContext?.state === 'suspended') {
    audioContext.resume().catch((error) => {
      console.error('効果音の再生を有効化できませんでした:', error)
    })
  }
}

function playSound(source, label) {
  loadAudioBuffer(source)
    .then((buffer) => {
      const context = audioContext
      if (!context || context.state === 'closed') {
        throw new Error('効果音用のAudioContextが利用できません。')
      }

      const resume = context.state === 'suspended' ? context.resume() : Promise.resolve()
      return resume.then(() => {
        if (context.state !== 'running') {
          throw new Error(`効果音用のAudioContextが再生状態ではありません (${context.state})。`)
        }

        const sourceNode = context.createBufferSource()
        sourceNode.buffer = buffer
        sourceNode.connect(context.destination)
        sourceNode.addEventListener('ended', () => activeAudioSources.delete(sourceNode), { once: true })
        activeAudioSources.add(sourceNode)
        sourceNode.start()
      })
    })
    .catch((error) => {
      console.error(`${label}を再生できませんでした:`, error)
  })
}

function playVoiceSound() {
  const audio = new Audio('/voice_sans.mp3')
  activeAudioPlayers.add(audio)
  audio.addEventListener('ended', () => activeAudioPlayers.delete(audio), { once: true })

  audio.play().catch((error) => {
    activeAudioPlayers.delete(audio)
    console.error('blue@walkerの音声を再生できませんでした:', error)
  })
}

function typeMessage(target, message, timerKey, onComplete = () => {}) {
  window.clearInterval(timerKey === 'blue' ? blueTypingTimer : enemyTypingTimer)
  const characters = Array.from(message)
  let index = 0
  target.value = ''

  const timer = window.setInterval(() => {
    target.value += characters[index]
    if (timerKey === 'blue') {
      playVoiceSound()
    }
    index += 1

    if (index >= characters.length) {
      window.clearInterval(timer)
      onComplete()
    }
  }, pacing.messageCharacter)

  if (timerKey === 'blue') {
    blueTypingTimer = timer
  } else {
    enemyTypingTimer = timer
  }
}

function answer(side) {
  if (resolved.value || gameFinished.value || !interfaceVisible.value) {
    return
  }

  resolved.value = true
  resultVisible.value = false
  messagesVisible.value = false
  const choice = side === 'left' ? currentQuestion.value.left : currentQuestion.value.right
  const correct = side === currentQuestion.value.answer
  if (!correct) {
    wrongAnswerCount.value += 1
  }
  const damageByMiss = [50, 20, 15, 10, 4]
  const damage = correct ? 0 : damageByMiss[wrongAnswerCount.value - 1]
  const nextHp = correct ? hp.value : Math.max(1, hp.value - damage)
  const actualDamage = hp.value - nextHp

  result.value = correct ? 'evaded' : 'hit'
  resultTimer = window.setTimeout(() => {
    resultVisible.value = true
    if (!correct && actualDamage > 0) {
      hp.value = nextHp
      playSound('/undertale-damage-taken.mp3', 'ダメージ音')
    } else if (correct) {
      playSound('/undertale-ding.mp3', '正解音')
    }

    messageDelayTimer = window.setTimeout(() => {
      messagesVisible.value = true

      if (correct) {
        typeMessage(
          blueMessage,
          `${choice}を選択。判断は正確だ。弾丸を回避した。`,
          'blue',
          () => {
            advanceTimer = window.setTimeout(advance, pacing.nextQuestionDelay)
          }
        )
        typeMessage(enemyMessage, 'くっ、弾丸をかわしたか。次は外さん。', 'enemy')
      } else {
        typeMessage(
          blueMessage,
          `${choice}を選択。被弾した！HPが${actualDamage}減少。`,
          'blue',
          () => {
            advanceTimer = window.setTimeout(advance, pacing.nextQuestionDelay)
          }
        )
        typeMessage(
          enemyMessage,
          `命中。HPを${actualDamage}削った。残りHP ${nextHp}。`,
          'enemy'
        )
      }
    }, pacing.resultMessageDelay)
  }, pacing.resultReveal)
}

function advance() {
  if (questionIndex.value === questions.length - 1) {
    gameFinished.value = true
    pendingChoice.value = null
    typeMessage(
      blueMessage,
      hp.value === 1
        ? '全問終了。HPは1。ぎりぎりで持ちこたえた。'
        : `全問終了。HP ${hp.value}で生還した。`,
      'blue'
    )
    typeMessage(enemyMessage, '標的を見失った。作戦を中止する。', 'enemy')
    return
  }

  questionIndex.value += 1
  pendingChoice.value = null
  selectionPending.value = false
  animationStarted.value = false
  resolved.value = false
  resultVisible.value = false
  result.value = null
  startCountdown()
  typeMessage(blueMessage, '次の問題だ。慎重に選べ。', 'blue')
  typeMessage(enemyMessage, '次の弾丸を装填中。標的を追跡している。', 'enemy')
}

function handleKeydown(event) {
  if (!interfaceVisible.value || selectionPending.value || resolved.value || gameFinished.value) {
    return
  }

  if (event.key === 'ArrowLeft') {
    event.preventDefault()
    if (event.repeat) return
    unlockAudio()
    selectAnswer('left')
  } else if (event.key === 'ArrowRight') {
    event.preventDefault()
    if (event.repeat) return
    unlockAudio()
    selectAnswer('right')
  }
}

function selectAnswer(side) {
  const selectedDuringCountdown = countdownValue.value !== null
  pendingChoice.value = side
  selectionPending.value = true
  selectionTimer = window.setTimeout(() => {
    animationStarted.value = true
    countdownValue.value = null
    window.clearTimeout(countdownStartTimer)
    window.clearInterval(countdownTimer)
    playSound('/カーソル移動5.mp3', 'カーソル移動音')
    if (selectedDuringCountdown) {
      bombTimer = window.setTimeout(() => {
        playSound('/undertale-bomb-explosion.mp3', '爆発音')
      }, 1500)
    }
    answer(side)
  }, pacing.choiceAnimation)
}

onMounted(() => {
  const AudioContextConstructor = window.AudioContext
  if (!AudioContextConstructor) {
    console.error('このブラウザーは効果音再生用のWeb Audio APIに対応していません。')
  } else {
    audioContext = new AudioContextConstructor()
    for (const source of soundEffects) {
      loadAudioBuffer(source).catch((error) => {
        console.error(`効果音を読み込めませんでした (${source}):`, error)
      })
    }
  }

  typeMessage(blueMessage, 'シミュレーション起動に成功しました。', 'blue')
  introTimer = window.setTimeout(() => {
    interfaceVisible.value = true
    messagesVisible.value = true
    startCountdown()
    typeMessage(blueMessage, 'リバイバル社の幹部を捕捉。気をつけて。', 'blue')
  }, pacing.interfaceIntro)
  window.addEventListener('keydown', handleKeydown)
})

onBeforeUnmount(() => {
  window.clearTimeout(introTimer)
  window.clearTimeout(advanceTimer)
  window.clearTimeout(resultTimer)
  window.clearTimeout(messageDelayTimer)
  window.clearTimeout(selectionTimer)
  window.clearTimeout(bombTimer)
  window.clearTimeout(countdownStartTimer)
  window.clearInterval(countdownTimer)
  window.clearInterval(blueTypingTimer)
  window.clearInterval(enemyTypingTimer)
  for (const audio of activeAudioPlayers) {
    audio.pause()
    audio.removeAttribute('src')
  }
  activeAudioPlayers.clear()
  for (const source of activeAudioSources) {
    source.stop()
  }
  activeAudioSources.clear()
  if (audioContext && audioContext.state !== 'closed') {
    audioContext.close().catch((error) => {
      console.error('効果音用AudioContextを終了できませんでした:', error)
    })
  }
  window.removeEventListener('keydown', handleKeydown)
})
</script>

<template>
  <section
    class="game-stage"
    :class="{
      'tilt-left': animationStarted && pendingChoice === 'left',
      'tilt-right': animationStarted && pendingChoice === 'right'
    }"
  >
    <header v-if="interfaceVisible" class="hp-panel">
      <div
        class="hp-track"
        role="meter"
        aria-label="プレイヤーHP"
        :aria-valuenow="hp"
        aria-valuemin="1"
        aria-valuemax="100"
      >
        <span class="hp-fill" :class="{ critical: hp <= 20 }" :style="{ width: `${hp}%` }"></span>
      </div>
    </header>

    <div v-if="interfaceVisible" class="game-layout">
      <main class="quiz-area">
        <div class="question-heading">
          <span class="question-number">QUESTION {{ questionIndex + 1 }} / 5</span>
        </div>

        <div
          v-if="countdownValue !== null"
          class="countdown-overlay"
          aria-live="off"
          aria-hidden="true"
        >
          {{ countdownValue }}
        </div>

        <QuestionBoard
          :key="questionIndex"
          :text="currentQuestion.text"
        />

        <div class="answer-options" :class="movingClass">
          <div :class="{ 'is-dismissed': resolved && pendingChoice !== 'left' }">
            <ChoiceButton
              side="left"
              :label="currentQuestion.left"
              :disabled="resolved"
              :selected="pendingChoice === 'left'"
              :dismissed="resolved && pendingChoice !== 'left'"
            />
          </div>
          <div :class="{ 'is-dismissed': resolved && pendingChoice !== 'right' }">
            <ChoiceButton
              side="right"
              :label="currentQuestion.right"
              :disabled="resolved"
              :selected="pendingChoice === 'right'"
              :dismissed="resolved && pendingChoice !== 'right'"
            />
          </div>
        </div>

        <div
          v-if="resultVisible"
          class="answer-result"
          :class="`is-${result}`"
          :aria-label="result === 'evaded' ? '正解' : '不正解'"
          aria-live="assertive"
          role="status"
        >
          <span class="result-symbol" aria-hidden="true">{{ result === 'evaded' ? '○' : '×' }}</span>
        </div>
      </main>

      <aside
        class="enemy-panel"
        :class="[{ 'is-message-hidden': !messagesVisible }, result ? `is-${result}` : '']"
      >
        <h2>リバイバル社 幹部</h2>
        <p class="enemy-status">{{ enemyMessage }}</p>
      </aside>

      <section class="blue-panel" :class="{ 'is-message-hidden': !messagesVisible }" aria-live="polite">
        <h2>blue@walker</h2>
        <p>{{ blueMessage }}<span class="typing-cursor">▋</span></p>
      </section>
    </div>

    <section v-else class="intro-panel" aria-live="polite">
      <h2>blue@walker</h2>
      <p>{{ blueMessage }}<span class="typing-cursor">▋</span></p>
    </section>
  </section>
</template>

<style scoped>
.game-stage {
  display: flex;
  width: min(100%, 1440px);
  min-height: calc(100svh - 72px);
  flex-direction: column;
  justify-content: center;
  gap: clamp(14px, 2vh, 24px);
  transform: rotate(0deg);
  transition: transform 700ms cubic-bezier(0.2, 0.8, 0.2, 1);
}

.game-stage.tilt-left {
  transform: rotate(-1.2deg);
}

.game-stage.tilt-right {
  transform: rotate(1.2deg);
}

.hp-panel {
  width: 100%;
  padding: 12px 18px;
  border: 1px solid var(--green-dim);
  background: rgba(2, 7, 4, 0.9);
  text-align: left;
}

.hp-track {
  height: 12px;
  overflow: hidden;
  border: 1px solid rgba(112, 255, 139, 0.35);
  background: #101711;
}

.hp-fill {
  display: block;
  height: 100%;
  background: linear-gradient(90deg, #258b42, var(--green));
  transition: width 700ms ease, background 200ms ease;
}

.hp-fill.critical {
  background: #ff5a5a;
}

.game-layout {
  display: grid;
  grid-template-columns: minmax(0, 1fr) minmax(230px, 290px);
  align-items: stretch;
  gap: 18px;
  min-height: 0;
}

.quiz-area {
  position: relative;
  display: flex;
  min-width: 0;
  flex-direction: column;
  justify-content: center;
  gap: clamp(12px, 2vh, 22px);
}

.countdown-overlay {
  position: absolute;
  z-index: 3;
  top: 50%;
  left: 50%;
  color: var(--green-bright);
  font-family: var(--mono);
  font-size: clamp(8rem, 24vw, 18rem);
  font-weight: 800;
  line-height: 1;
  text-shadow: 0 0 40px rgba(112, 255, 139, 0.7);
  transform: translate(-50%, -50%);
  animation: countdown-pop 240ms ease-out both;
  pointer-events: none;
}

@keyframes countdown-pop {
  from {
    opacity: 0;
    scale: 0.75;
  }

  to {
    opacity: 1;
    scale: 1;
  }
}

.question-heading {
  display: flex;
  justify-content: space-between;
  color: var(--muted);
  font-family: var(--mono);
  font-size: 0.68rem;
  letter-spacing: 0.1em;
}

.answer-options {
  --option-gap: clamp(12px, 2vw, 22px);
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: var(--option-gap);
}

.answer-options > div {
  aspect-ratio: 1;
  min-width: 0;
  transition: transform 700ms cubic-bezier(0.2, 0.8, 0.2, 1), opacity 700ms ease;
}

.answer-options :deep(button) {
  height: 100%;
  min-height: 0;
}

.answer-options.move-right > div:first-child {
  transform: translateX(52%);
  z-index: 1;
}

.answer-options.move-right > div:last-child {
  transform: translateX(110%);
  opacity: 0;
}

.answer-options.move-left > div:first-child {
  transform: translateX(-110%);
  opacity: 0;
}

.answer-options.move-left > div:last-child {
  transform: translateX(-52%);
  z-index: 1;
}

.answer-result {
  position: absolute;
  z-index: 2;
  top: 50%;
  left: 50%;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  transform: translate(-50%, -50%);
  animation: result-pop 320ms cubic-bezier(0.2, 0.8, 0.2, 1) both;
  pointer-events: none;
  text-shadow: 0 0 32px currentColor;
}

.answer-result.is-evaded {
  color: var(--green-bright);
}

.answer-result.is-hit {
  color: #ff6060;
}

.result-symbol {
  font-family: var(--mono);
  font-size: clamp(8rem, 22vw, 16rem);
  font-weight: 800;
  line-height: 1;
}

@keyframes result-pop {
  from {
    opacity: 0;
    scale: 0.6;
  }

  to {
    opacity: 1;
    scale: 1;
  }
}

.enemy-panel {
  display: flex;
  grid-column: 2;
  grid-row: 1 / span 2;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  padding: 12px;
  color: #f1dada;
  text-align: center;
}

.is-message-hidden {
  visibility: hidden;
}

.enemy-panel h2 {
  margin: 0;
  color: #ff7777;
  font-size: clamp(0.85rem, 1.4vw, 1rem);
}

.enemy-status {
  margin: 0;
  font-family: var(--mono);
  font-size: 0.82rem;
  line-height: 1.8;
  overflow-wrap: anywhere;
}

.blue-panel,
.intro-panel {
  width: 100%;
  min-height: clamp(150px, 20vh, 210px);
  padding: 18px 22px;
  border: 1px solid var(--green-dim);
  background: rgba(1, 5, 2, 0.97);
  text-align: center;
}

.blue-panel {
  grid-column: 1;
}

.blue-panel h2,
.intro-panel h2 {
  margin: 0 0 14px;
  color: var(--green-bright);
  font-family: var(--mono);
  font-size: 0.9rem;
  letter-spacing: 0.08em;
}

.blue-panel p,
.intro-panel p {
  display: grid;
  min-height: 80px;
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

.intro-panel {
  display: flex;
  flex-direction: column;
  justify-content: center;
  min-height: min(420px, 58vh);
}

.intro-panel p {
  white-space: pre-wrap;
}

@keyframes blink {
  to {
    visibility: hidden;
  }
}

@media (max-width: 800px) {
  .game-layout {
    grid-template-columns: minmax(0, 1fr) minmax(170px, 220px);
  }
}

@media (max-width: 620px) {
  .game-stage {
    min-height: calc(100svh - 48px);
    gap: 10px;
  }

  .game-layout {
    grid-template-columns: 1fr;
    gap: 10px;
  }

  .enemy-panel {
    grid-column: 1;
    grid-row: auto;
    min-height: 0;
    padding: 4px 8px;
  }

  .answer-options > div {
    min-height: 0;
  }

  .enemy-panel {
    min-height: 100px;
    gap: 8px;
    padding: 10px;
  }

  .enemy-status {
    font-size: 0.72rem;
  }

  .blue-panel,
  .intro-panel {
    min-height: 120px;
    padding: 12px;
  }

  .blue-panel p,
  .intro-panel p {
    min-height: 60px;
    font-size: 0.95rem;
  }

}
</style>
