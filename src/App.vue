<script setup lang="ts">
import { ref } from 'vue'
import _data from './words.json'

class Game {
  _words: string[]
  _used_words: string[]
  _wins: string[]
  _losses: string[]
  constructor(words: string[] = []) {
    this._words = words ?? []
    this._used_words = []
    this._wins = []
    this._losses = []
  }
  get _available_words() {
    return this._words.filter((w) => !this._used_words.includes(w))
  }
  getWinsCount() {
    return this._wins.length
  }
  getLossesCount() {
    return this._losses.length
  }
  loadData() {
    const _localData = window.localStorage.getItem('_game')
    if (!_localData) return
    const _savedGame = JSON.parse(_localData) as {
      wins: string[]
      losses: string[]
      used: string[]
    }
    this._losses = _savedGame.losses ?? []
    this._wins = _savedGame.wins ?? []
    this._used_words = _savedGame.used
  }
  saveData() {
    window.localStorage.setItem(
      '_game',
      JSON.stringify({
        used: this._used_words,
        wins: this._wins,
        losses: this._losses
      })
    )
  }
  addToWin(word: string) {
    this._wins.push(word)
    this._addToUsed(word)
  }
  addToLosses(word: string) {
    this._losses.push(word)
    this._addToUsed(word)
  }
  _addToUsed(word: string) {
    this._used_words.push(word)
  }
  getNewWord() {
    return this._available_words[Math.floor(Math.random() * this._available_words.length)]
  }
}

const game = new Game(_data)
game.loadData()
console.log('game: ', new Set(_data))

const activeWord = ref('')
const status = ref<'wrong' | 'right' | null>(null)
const isStart = ref(true)

function setStatus(newStatus: 'wrong' | 'right') {
  if (isStart.value) return
  if (newStatus === status.value) {
    status.value = null
  } else {
    status.value = newStatus
  }
}

const wordDOM = ref<HTMLElement>()

function nextWord() {
  if (isStart.value) {
    isStart.value = false
  }
  if (activeWord.value !== '') {
    status.value === 'right' ? game.addToWin(activeWord.value) : game.addToLosses(activeWord.value)
  }
  activeWord.value = game.getNewWord()
  status.value = null
  game.saveData()
}
</script>

<template>
  <main>
    <div ref="wordDOM" class="word-wrapper">
      <p v-if="false" class="available">{{}}</p>
      <p class="word">
        {{ activeWord }}
      </p>
    </div>
    <div class="navigation">
      <div class="button-wrapper" :class="{ active: status !== null || isStart }">
        <button class="button" @click="nextWord">Наступне слово</button>
      </div>
      <div class="statuses">
        <button
          class="status status__error"
          @click="() => setStatus('wrong')"
          :class="{ active: status === 'wrong' }"
        />
        <button
          class="status status__success"
          @click="() => setStatus('right')"
          :class="{ active: status === 'right' }"
        />
      </div>
    </div>
  </main>
</template>

<style scoped>
.word-wrapper {
  position: fixed;
  left: 0;
  top: 0;
  right: 0;
  bottom: 0;
  display: flex;
  justify-content: center;
  align-items: center;
}
.available {
  position: fixed;
  left: 1rem;
  top: 1rem;
  font-size: 1rem;
  line-height: 1.2;
  color: var(--c-dark);
}
.word {
  font-weight: 600;
  font-size: 2rem;
  text-transform: uppercase;
  letter-spacing: -0.04em;
}
.navigation {
  display: flex;
  flex-direction: column;
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.statuses {
  display: grid;
  grid-template-columns: 1fr 1fr;
  border-radius: 1rem 1rem 0 0;
  overflow: hidden;
  isolation: isolate;
  width: 100%;
}
.status {
  display: block;
  border: none;
  outline: none;
  height: 4rem;
  background: var(--button-color);
  transition-duration: var(--transition-duration, 0.4s);
  transition-timing-function: var(--transition-easing, ease-out);
  opacity: 0.4;
}
.status.active {
  opacity: 1;
}
.status__error {
  --button-color: var(--c-red);
}
.status__success {
  --button-color: var(--c-green);
}

/* button */
.button-wrapper {
  margin-bottom: 2rem;
  transition-duration: var(--transition-duration, 0.4s);
  transition-timing-function: var(--transition-easing, ease-out);
  pointer-events: none;
  opacity: 0;
}

.button-wrapper.active {
  opacity: 1;
  pointer-events: all;
}

.button {
  display: flex;
  justify-content: center;
  align-items: center;
  border: none;
  outline: none;
  padding: 0 2rem;
  height: 4rem;
  color: white;
  border-radius: 0.5rem;
  background: var(--c-dark);
  font-size: 1rem;
  line-height: 1;
  text-transform: uppercase;
}
</style>
../data/words.js
