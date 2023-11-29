<script setup lang="ts">
import GameHeader from './components/GameHeader.vue'
import GameFigure from './components/GameFigure.vue'
import GameWrong from './components/GameWrong.vue'
import GameWord from './components/GameWord.vue'
import GamePopup from './components/GamePopup.vue'
import GameNotification from './components/GameNotification.vue'
import { computed, ref, watch } from 'vue'
import { getRandomName } from './api/getRandomName'

const word = ref('')
const getName = async () => {
  try {
    const name = await getRandomName()
    word.value = name.toLowerCase()
  } catch (err) {
    console.log(err)
    word.value = ''
  }
}
getName()

const letters = ref<string[]>([])
const correctLetters = computed(() => letters.value.filter((i) => word.value.includes(i)))
const wrongLetters = computed(() => letters.value.filter((i) => !word.value.includes(i)))
const notification = ref<InstanceType<typeof GameNotification> | null>(null)
const popup = ref<InstanceType<typeof GamePopup> | null>(null)
const isLose = computed(() => wrongLetters.value.length === 6)
const isWin = computed(() => [...word.value].every((x) => correctLetters.value.includes(x)))

watch(wrongLetters, () => {
  if (isLose.value) {
    popup.value?.open('lose')
  }
})

watch(correctLetters, () => {
  if (isWin.value) {
    popup.value?.open('win')
  }
})

window.addEventListener('keydown', ({ key }) => {
  if (isLose.value || isWin.value) {
    return
  }
  if (letters.value.includes(key)) {
    notification.value?.open()
    setTimeout(() => notification.value?.close(), 2000)
    return
  }
  if (/[а-яА-ЯёЁ]/.test(key)) {
    letters.value.push(key.toLowerCase())
  }
})
const restart = async () => {
  await getName()
  letters.value = []
  popup.value?.close()
}
</script>

<template>
  <GameHeader />
  <div class="game-container">
    <GameFigure :wrong-letters-count="wrongLetters.length" />

    <GameWrong :wrong-letters="wrongLetters" />

    <GameWord :word="word" :correct-letters="correctLetters" />
  </div>

  <!-- Container for final message -->
  <GamePopup ref="popup" :word="word" @restart="restart" />

  <!-- Notification -->
  <GameNotification ref="notification" />
</template>
