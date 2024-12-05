<template>
  <div
    :class="[
      'min-h-screen flex items-center justify-center',
      darkMode ? 'bg-gray-800' : 'bg-gray-100'
    ]"
  >
    <div
      :class="['w-full max-w-3xl p-8 rounded-lg shadow-lg', darkMode ? 'bg-gray-700' : 'bg-white']"
    >
      <h1 class="text-4xl font-bold text-center mb-8" :class="{ 'text-white': darkMode }">
        Typing Speed Tester
      </h1>
      <button
        @click="toggleDarkMode"
        class="absolute top-4 right-4 bg-gray-200 rounded-full p-2 hover:bg-gray-300"
      >
        {{ darkMode ? 'Light Mode' : 'Dark Mode' }}
      </button>

      <div v-if="!started">
        <div class="mb-4">
          <label for="language" class="block mb-2">Select Language:</label>
          <select v-model="selectedLanguage" id="language" class="w-full p-2 border rounded-lg">
            <option value="english">English</option>
            <option value="indonesian">Bahasa Indonesia</option>
          </select>
        </div>
        <div class="mb-4">
          <label for="duration" class="block mb-2">Select Test Duration:</label>
          <select v-model="selectedDuration" id="duration" class="w-full p-2 border rounded-lg">
            <option v-for="duration in durations" :key="duration" :value="duration">
              {{ duration }} seconds
            </option>
          </select>
        </div>
        <div class="mb-4">
          <label for="text" class="block mb-2">Select Sample Text:</label>
          <select v-model="selectedText" id="text" class="w-full p-2 border rounded-lg">
            <option v-for="(text, index) in filteredSampleTexts" :key="index" :value="text">
              {{ text.slice(0, 30) }}...
            </option>
          </select>
        </div>
        <button
          @click="startTest"
          class="w-full bg-blue-600 text-white py-3 px-6 rounded-lg hover:bg-blue-700 text-xl"
        >
          Start Test
        </button>
        <button
          @click="showTutorial"
          class="mt-4 w-full bg-green-600 text-white py-3 px-6 rounded-lg hover:bg-green-700 text-xl"
        >
          Show Tutorial
        </button>
        <button
          @click="showStatistics"
          class="mt-4 w-full bg-gray-600 text-white py-3 px-6 rounded-lg hover:bg-gray-700 text-xl"
        >
          Show Detailed Statistics
        </button>
        <p v-if="!started && showStats" class="mt-4 text-center text-red-500">
          Your statistics are empty because you haven't tried the test yet.
        </p>
      </div>

      <div v-else>
        <div class="flex justify-between items-center mb-4">
          <p class="text-lg font-semibold" :class="{ 'text-white': darkMode }">Time: {{ time }}s</p>
          <p class="text-lg font-semibold" :class="{ 'text-white': darkMode }">
            Errors: {{ errors }}
          </p>
        </div>
        <div class="mb-6 p-4 bg-gray-100 rounded-lg" :class="{ 'bg-gray-600': darkMode }">
          <p class="text-xl leading-relaxed" v-html="highlightedText"></p>
        </div>
        <textarea
          v-model="userInput"
          @input="checkInput"
          :placeholder="!paused ? 'Start typing here...' : ''"
          class="w-full h-64 p-4 border rounded-lg focus:outline-none focus:border-blue-500 text-lg"
          :disabled="!started || time === 0"
          :class="{ 'bg-gray-800 text-white': darkMode }"
        ></textarea>
        <div class="mt-4 flex justify-between">
          <button
            @click="pauseResumeTest"
            class="bg-yellow-500 text-white py-2 px-4 rounded-lg hover:bg-yellow-600"
          >
            {{ paused ? 'Resume' : 'Pause' }}
          </button>
          <div class="flex space-x-4">
            <button
              @click="resetTest"
              class="bg-red-500 text-white py-2 px-4 rounded-lg hover:bg-red-600"
            >
              Reset
            </button>
            <button
              @click="saveStatistics"
              class="bg-purple-600 text-white py-2 px-4 rounded-lg hover:bg-purple-700"
            >
              Done
            </button>
          </div>
        </div>
        <div v-if="time === 0" class="mt-6 text-center">
          <p class="text-2xl font-bold mb-4" :class="{ 'text-white': darkMode }">Your Result:</p>
          <p class="text-2xl font-bold mb-4" :class="{ 'text-white': darkMode }">
            Your final typing speed is {{ typingSpeed }} WPM
          </p>
          <button
            @click="showStatistics"
            class="mt-4 w-full bg-blue-600 text-white py-3 px-6 rounded-lg hover:bg-blue-700 text-xl"
          >
            Show Detailed Statistics
          </button>
          <div v-if="showStats" class="mt-4">
            <h3 class="text-xl font-semibold" :class="{ 'text-white': darkMode }">
              Detailed Statistics:
            </h3>
            <ul class="list-disc list-inside text-lg" :class="{ 'text-white': darkMode }">
              <li>Words Typed: {{ totalWords }}</li>
              <li>Errors Made: {{ errors }}</li>
              <li>Accuracy: {{ accuracy }}%</li>
              <li>Time Taken: {{ selectedDuration - time }} seconds</li>
              <li>Average Speed: {{ typingSpeed }} WPM</li>
            </ul>
          </div>
          <button
            @click="resetTest"
            class="w-full bg-green-600 text-white py-3 px-6 rounded-lg hover:bg-green-700 text-xl"
          >
            Restart
          </button>
        </div>
        <div v-else class="mt-6 text-center">
          <p class="text-2xl font-bold" :class="{ 'text-white': darkMode }">
            Current Speed: {{ currentSpeed }} WPM
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      darkMode: false,
      sampleTexts: {
        english: [
          'The quick brown fox jumps over the lazy dog.',
          'Pack my box with five dozen liquor jugs.',
          'How quickly daft jumping zebras vex.'
        ],
        indonesian: [
          'Pagi hari yang cerah di sebuah desa kecil.',
          'Bulan purnama bersinar terang di langit malam.',
          'Di tengah hutan yang lebat, terdengar suara burung berkicau.'
        ]
      },
      selectedLanguage: 'english',
      selectedText: '',
      userInput: '',
      started: false,
      paused: false,
      time: 60,
      selectedDuration: 60,
      timer: null,
      typingSpeed: 0,
      errors: 0,
      currentSpeed: 0,
      totalWords: 0,
      accuracy: 0,
      durations: [30, 60, 120],
      showStats: false // State to track if statistics should be shown
    }
  },
  computed: {
    filteredSampleTexts() {
      return this.sampleTexts[this.selectedLanguage]
    },
    highlightedText() {
      let highlighted = ''
      const enteredText = this.userInput
      for (let i = 0; i < this.selectedText.length; i++) {
        if (i < enteredText.length) {
          if (enteredText[i] === this.selectedText[i]) {
            highlighted += `<span class="text-green-500">${enteredText[i]}</span>`
          } else {
            highlighted += `<span class="text-red-500">${this.selectedText[i]}</span>`
          }
        } else {
          highlighted += `<span>${this.selectedText[i]}</span>`
        }
      }
      return highlighted
    }
  },
  methods: {
    startTest() {
      if (this.selectedText === '') {
        this.selectedText = this.filteredSampleTexts[0]
      }
      this.started = true
      this.userInput = ''
      this.time = this.selectedDuration
      this.errors = 0
      this.typingSpeed = 0
      this.currentSpeed = 0
      this.paused = false
      this.showStats = false // Reset statistics display
      this.timer = setInterval(() => {
        if (!this.paused && this.time > 0) {
          this.time--
          this.calculateCurrentSpeed()
        } else if (this.time === 0) {
          clearInterval(this.timer)
          this.calculateFinalSpeed()
        }
      }, 1000)
    },
    checkInput() {
      const enteredText = this.userInput
      const sampleTextSlice = this.selectedText.slice(0, enteredText.length)
      this.errors = enteredText
        .split('')
        .reduce((acc, char, i) => acc + (char !== sampleTextSlice[i] ? 1 : 0), 0)
      if (this.time > 0 && this.userInput === this.selectedText) {
        clearInterval(this.timer)
        this.calculateFinalSpeed()
      }
    },
    calculateCurrentSpeed() {
      const words = this.userInput
        .trim()
        .split(' ')
        .filter((word) => word.length > 0).length
      const minutes = (this.selectedDuration - this.time) / 60
      this.currentSpeed = minutes > 0 ? Math.round(words / minutes) : 0
    },
    calculateFinalSpeed() {
      const words = this.userInput
        .trim()
        .split(' ')
        .filter((word) => word.length > 0).length
      const minutes = this.selectedDuration / 60
      this.typingSpeed = Math.round(words / minutes)
      this.totalWords = words
      this.accuracy = words > 0 ? Math.round(((words - this.errors) / words) * 100) : 0
    },
    pauseResumeTest() {
      this.paused = !this.paused
      if (this.paused) {
        this.userInput = ''
        this.errors = 0
      }
    },
    resetTest() {
      clearInterval(this.timer)
      this.started = false
      this.paused = false
      this.userInput = ''
      this.errors = 0
      this.typingSpeed = 0
      this.currentSpeed = 0
      this.totalWords = 0
      this.accuracy = 0
      this.time = this.selectedDuration
      this.showStats = false // Reset statistics display
    },
    toggleDarkMode() {
      this.darkMode = !this.darkMode
    },
    showTutorial() {
      alert(
        'This is a typing speed tester. Select a language, duration, and sample text to start typing. Your speed, accuracy, and errors will be displayed at the end of the test.'
      )
    },
    showStatistics() {
      this.showStats = true // Show detailed statistics
    },
    saveStatistics() {
      const stats = {
        language: this.selectedLanguage,
        duration: this.selectedDuration,
        totalWords: this.totalWords,
        errors: this.errors,
        accuracy: this.accuracy,
        typingSpeed: this.typingSpeed
      }
      localStorage.setItem('typingStats', JSON.stringify(stats))
      alert('Statistics saved successfully!')
    }
  },
  mounted() {
    // Load existing statistics from Local Storage on mount
    const savedStats = localStorage.getItem('typingStats')
    if (savedStats) {
      const stats = JSON.parse(savedStats)
      this.totalWords = stats.totalWords || 0
      this.errors = stats.errors || 0
      this.accuracy = stats.accuracy || 0
      this.typingSpeed = stats.typingSpeed || 0
    }
  }
}
</script>

<style>
body {
  font-family: 'Inter', sans-serif;
}
</style>
