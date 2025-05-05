<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'

const quotes = [
  'The future belongs to those who believe in the beauty of their dreams. - Eleanor Roosevelt',
  'Any sufficiently advanced technology is indistinguishable from magic. - Arthur C. Clarke',
  'The best way to predict the future is to invent it. - Alan Kay',
  'Technology is a useful servant but a dangerous master. - Christian Lous Lange',
  "The advance of technology is based on making it fit in so that you don't really even notice it. - Bill Gates",
]

type TestState = 'waiting' | 'running' | 'finished'

const state = ref<TestState>('waiting')
const inputText = ref('')
const startTime = ref(0)
const endTime = ref(0)
const currentQuote = ref('')
const cursorPosition = ref(0)

// Track all keystrokes and their correctness
const keyStrokes = ref<{ key: string; correct: boolean; timestamp: number }[]>([])

const wpm = computed(() => {
  if (endTime.value === 0) return 0
  const minutes = (endTime.value - startTime.value) / 60000
  const words = inputText.value.trim().split(/\s+/).length
  return Math.round(words / minutes)
})

const accuracy = computed(() => {
  if (keyStrokes.value.length === 0) return 0

  const correctStrokes = keyStrokes.value.filter((k) => k.correct).length
  const totalStrokes = keyStrokes.value.length

  return Math.round((correctStrokes / totalStrokes) * 100)
})

const progress = computed(() => {
  if (!currentQuote.value) return 0
  return Math.min(100, (cursorPosition.value / currentQuote.value.length) * 100)
})

const startTest = () => {
  state.value = 'running'
  inputText.value = ''
  startTime.value = Date.now()
  endTime.value = 0
  cursorPosition.value = 0
  keyStrokes.value = []
  currentQuote.value = quotes[Math.floor(Math.random() * quotes.length)]
}

const handleInput = (e: Event) => {
  const target = e.target as HTMLTextAreaElement
  const newValue = target.value
  const newPosition = target.selectionStart || 0

  // Detect if this is a backspace (deletion)
  if (newValue.length < inputText.value.length) {
    // Remove the last keystroke from our records
    keyStrokes.value.pop()
  } else {
    // Track each new keystroke and its correctness
    const newChar = newValue[newPosition - 1]
    const expectedChar = currentQuote.value[newPosition - 1]
    const isCorrect = newChar === expectedChar

    keyStrokes.value.push({
      key: newChar,
      correct: isCorrect,
      timestamp: Date.now(),
    })
  }

  inputText.value = newValue
  cursorPosition.value = newPosition

  // Check if test is complete
  if (inputText.value === currentQuote.value) {
    endTest()
  }
}

const endTest = () => {
  endTime.value = Date.now()
  state.value = 'finished'
}

const resetTest = () => {
  state.value = 'waiting'
}

onMounted(() => {
  const textarea = document.querySelector('textarea')
  if (textarea) {
    textarea.focus()
  }
})
</script>

<template>
  <div class="typing-test" :class="state">
    <div class="neon-border"></div>

    <div class="header">
      <h1>FUTURE TYPE</h1>
      <p class="subtitle">Test your typing speed in the digital age</p>
    </div>

    <div class="quote-display" v-if="state !== 'waiting'">
      <span
        v-for="(char, index) in currentQuote"
        :key="index"
        :class="{
          correct: index < inputText.length && inputText[index] === char,
          incorrect: index < inputText.length && inputText[index] !== char,
          current: index === cursorPosition,
          untyped: index >= inputText.length,
        }"
      >
        {{ char }}
      </span>
    </div>

    <div class="controls">
      <button v-if="state === 'waiting'" @click="startTest" class="neon-button">START TEST</button>

      <textarea
        v-if="state === 'running'"
        v-model="inputText"
        @input="handleInput"
        spellcheck="false"
        autofocus
      ></textarea>

      <button v-if="state === 'finished'" @click="resetTest" class="neon-button">TRY AGAIN</button>
    </div>

    <div class="progress-bar">
      <div class="progress" :style="{ width: `${progress}%` }"></div>
    </div>

    <div class="stats" v-if="state === 'finished'">
      <div class="stat">
        <div class="stat-value">{{ wpm }}</div>
        <div class="stat-label">WPM</div>
      </div>
      <div class="stat">
        <div class="stat-value">{{ accuracy }}%</div>
        <div class="stat-label">ACCURACY</div>
      </div>
      <div class="stat">
        <div class="stat-value">{{ Math.round((endTime - startTime) / 1000) }}</div>
        <div class="stat-label">SECONDS</div>
      </div>
    </div>

    <div class="instructions" v-if="state === 'waiting'">
      <p>Press START TEST to begin</p>
      <p>Type the displayed text as fast as you can</p>
      <p>The test ends when you complete the entire text</p>
    </div>
  </div>
</template>

<style scoped>
.typing-test {
  position: relative;
  width: 100%;
  max-width: 800px;
  padding: 2rem;
  background: rgba(10, 10, 26, 0.8);
  border-radius: 10px;
  box-shadow: 0 0 20px rgba(0, 255, 255, 0.1);
  overflow: hidden;
}

.neon-border {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 3px;
  background: linear-gradient(90deg, transparent, #0ff, transparent);
  animation: borderGlow 2s linear infinite;
}

@keyframes borderGlow {
  0% {
    opacity: 0;
  }
  50% {
    opacity: 1;
  }
  100% {
    opacity: 0;
  }
}

.header {
  text-align: center;
  margin-bottom: 2rem;
}

h1 {
  font-size: 3rem;
  font-weight: bold;
  background: linear-gradient(to right, #0ff, #f0f);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  text-shadow: 0 0 10px rgba(0, 255, 255, 0.3);
  letter-spacing: 2px;
  margin-bottom: 0.5rem;
}

.subtitle {
  color: #a0a0ff;
  font-size: 0.9rem;
  letter-spacing: 1px;
}

.quote-display {
  font-size: 1.2rem;
  line-height: 1.6;
  margin-bottom: 2rem;
  min-height: 120px;
  background: rgba(20, 20, 40, 0.5);
  padding: 1rem;
  border-radius: 5px;
  border-left: 2px solid #0ff;
}

.correct {
  color: #0f0;
}

.incorrect {
  color: #f00;
  text-decoration: underline;
}

.current {
  background: rgba(0, 255, 255, 0.3);
  position: relative;
}

.current::after {
  content: '';
  position: absolute;
  bottom: -2px;
  left: 0;
  width: 100%;
  height: 2px;
  background: #0ff;
  animation: cursorBlink 1s infinite;
}

@keyframes cursorBlink {
  0%,
  100% {
    opacity: 1;
  }
  50% {
    opacity: 0;
  }
}

.untyped {
  color: #a0a0ff;
}

.controls {
  margin-bottom: 2rem;
  text-align: center;
}

textarea {
  width: 100%;
  height: 100px;
  background: rgba(20, 20, 40, 0.8);
  border: 1px solid #0ff;
  border-radius: 5px;
  padding: 1rem;
  color: #fff;
  font-family: 'Orbitron', sans-serif;
  font-size: 1rem;
  resize: none;
  outline: none;
  box-shadow: 0 0 10px rgba(0, 255, 255, 0.2);
}

.neon-button {
  background: transparent;
  border: 2px solid #0ff;
  color: #0ff;
  padding: 0.8rem 2rem;
  font-family: 'Orbitron', sans-serif;
  font-size: 1rem;
  cursor: pointer;
  border-radius: 5px;
  transition: all 0.3s;
  text-transform: uppercase;
  letter-spacing: 1px;
  box-shadow: 0 0 10px rgba(0, 255, 255, 0.3);
}

.neon-button:hover {
  background: rgba(0, 255, 255, 0.1);
  box-shadow: 0 0 20px rgba(0, 255, 255, 0.5);
  transform: translateY(-2px);
}

.progress-bar {
  height: 5px;
  background: rgba(20, 20, 40, 0.8);
  border-radius: 5px;
  margin-bottom: 2rem;
  overflow: hidden;
}

.progress {
  height: 100%;
  background: linear-gradient(90deg, #0ff, #f0f);
  transition: width 0.1s;
}

.stats {
  display: flex;
  justify-content: space-around;
  margin-bottom: 2rem;
}

.stat {
  text-align: center;
}

.stat-value {
  font-size: 2rem;
  font-weight: bold;
  color: #0ff;
  margin-bottom: 0.5rem;
}

.stat-label {
  font-size: 0.8rem;
  color: #a0a0ff;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.instructions {
  text-align: center;
  color: #a0a0ff;
  font-size: 0.9rem;
  line-height: 1.6;
}

@media (max-width: 768px) {
  h1 {
    font-size: 2rem;
  }

  .quote-display {
    font-size: 1rem;
  }

  .stat-value {
    font-size: 1.5rem;
  }
}
</style>
