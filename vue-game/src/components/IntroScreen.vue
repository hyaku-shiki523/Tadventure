<script setup>
import { onBeforeUnmount, onMounted, ref } from 'vue'

const emit = defineEmits(['complete', 'skip'])

const messages = [
  'はじめまして。\n別班のもとで協力しているハッカーのブルーウォーカーです。',
  '黒須さんは、、、そちらに乗っていますか！？',
  'そうですか...　黒須さんはそちらにはいないのですね...',
  'すみません、話を戻します。',
  'リバイバル社の地下のハッキングに成功しました。',
  'どうやら、リバイバル社の侵入者に対する戦闘モードが起動したようです。',
  'これから、あなたがこの地上につながる地下トンネルから脱出できるよう全力でサポートさせていただきます。',
  '２択クイズで正解することによって、リバイバル社の幹部からの５回の攻撃を回避することができます。',
  'あなたのHPゲージが上部、敵からの攻撃状況は右部、わたしとの通信は下部で行います。',
  'でも、攻撃システムがクイズだなんて...',
  '開発者は敵の攻撃システムについて悩みに頭を悩ませたのかしら...',
  'そんなことより、、、この戦闘システムについて理解したら、トロッコの右側に全員が寄って、右側にトロッコを傾けてください。もう一度、説明が必要な場合は、左側にトロッコを傾けてください。',
  'そうです！　戦闘システムについて理解していただけたようですね。',
  'すみません、少し長くなってしまいましたが、これで戦闘システムに説明は終わりです。',
  'シミュレーションのプログラムが完成しました。',
  '生きて帰ってきてくださいね...',
  'では、リバイバル社の戦闘システムのシミュレーションを開始します。'
]
const quizInstructionsIndex = 7
const hpInstructionsIndex = 8
const confirmationIndex = 11
const messageIndex = ref(0)
const visibleMessage = ref('')
const awaitingContinue = ref(false)
let repeatingInstructions = false
let typingTimer
let advanceTimer
const activeVoices = new Set()

function playVoiceSound() {
  const audio = new Audio('/voice_sans.mp3')
  activeVoices.add(audio)
  audio.addEventListener('ended', () => activeVoices.delete(audio), { once: true })
  audio.play().catch((error) => {
    activeVoices.delete(audio)
    console.error('blue@walkerの音声を再生できませんでした:', error)
  })
}

function typeCurrentMessage() {
  window.clearInterval(typingTimer)
  const characters = Array.from(messages[messageIndex.value])
  let characterIndex = 0
  visibleMessage.value = ''
  awaitingContinue.value = false

  typingTimer = window.setInterval(() => {
    visibleMessage.value += characters[characterIndex]
    playVoiceSound()
    characterIndex += 1

    if (characterIndex >= characters.length) {
      window.clearInterval(typingTimer)

      if (messageIndex.value === confirmationIndex) {
        awaitingContinue.value = true
      } else if (
        repeatingInstructions
        && messageIndex.value === hpInstructionsIndex
      ) {
        repeatingInstructions = false
        advanceTimer = window.setTimeout(() => showMessage(confirmationIndex), 1500)
      } else if (messageIndex.value === messages.length - 1) {
        advanceTimer = window.setTimeout(() => emit('complete'), 1500)
      } else {
        advanceTimer = window.setTimeout(showNextMessage, 1500)
      }
    }
  }, 30)
}

function showNextMessage() {
  showMessage(messageIndex.value + 1)
}

function showMessage(index) {
  messageIndex.value = index
  typeCurrentMessage()
}

function handleKeydown(event) {
  if (event.code === 'Space' && !event.repeat) {
    event.preventDefault()
    emit('skip')
    return
  }

  if (
    !['ArrowLeft', 'ArrowRight'].includes(event.key)
    || !awaitingContinue.value
    || event.repeat
  ) {
    return
  }

  event.preventDefault()
  awaitingContinue.value = false
  window.clearTimeout(advanceTimer)
  if (event.key === 'ArrowLeft') {
    repeatingInstructions = true
    showMessage(quizInstructionsIndex)
  } else {
    showNextMessage()
  }
}

onMounted(() => {
  typeCurrentMessage()
  window.addEventListener('keydown', handleKeydown)
})

onBeforeUnmount(() => {
  window.clearInterval(typingTimer)
  window.clearTimeout(advanceTimer)
  for (const audio of activeVoices) {
    audio.pause()
    audio.removeAttribute('src')
  }
  activeVoices.clear()
  window.removeEventListener('keydown', handleKeydown)
})
</script>

<template>
  <section class="intro-panel" aria-label="ブルーウォーカーからの通信">
    <h1>blue@walker</h1>
    <p class="message">
      <span>{{ visibleMessage }}<span class="typing-cursor">▋</span></span>
    </p>
  </section>
</template>

<style scoped>
.intro-panel {
  position: relative;
  display: flex;
  width: min(100%, 900px);
  min-height: min(460px, 68vh);
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: clamp(28px, 6vw, 64px);
  border: 1px solid var(--green-dim);
  background:
    linear-gradient(110deg, rgba(112, 255, 139, 0.06), transparent 42%),
    rgba(1, 5, 2, 0.97);
  box-shadow: 0 20px 70px rgba(0, 0, 0, 0.48), inset 0 0 42px rgba(112, 255, 139, 0.035);
  text-align: center;
}

h1 {
  margin: 0 0 30px;
  color: var(--green-bright);
  font-family: var(--mono);
  font-size: clamp(1rem, 2vw, 1.25rem);
  letter-spacing: 0.1em;
}

.message {
  display: grid;
  min-height: 110px;
  place-items: center;
  margin: 0;
  color: var(--text);
  font-family: var(--mono);
  font-size: clamp(1rem, 2.4vw, 1.45rem);
  line-height: 2;
  white-space: pre-wrap;
}

.typing-cursor {
  display: inline-block;
  margin-left: 3px;
  color: var(--green);
  animation: blink 0.8s steps(2, start) infinite;
}

@keyframes blink {
  to {
    visibility: hidden;
  }
}

@media (max-width: 520px) {
  .intro-panel {
    min-height: min(420px, 72vh);
    padding: 48px 20px;
  }

  .message {
    min-height: 150px;
    font-size: 0.95rem;
  }
}
</style>
