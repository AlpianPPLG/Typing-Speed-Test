<template>
  <div class="min-h-screen flex items-center justify-center bg-gray-100">
    <div class="w-full max-w-3xl p-8 bg-white rounded-lg shadow-lg">
      <h1 class="text-4xl font-bold text-center mb-8">Typing Speed Tester</h1>
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
      </div>
      <div v-else>
        <div class="flex justify-between items-center mb-4">
          <p class="text-lg font-semibold">Time: {{ time }}s</p>
          <p class="text-lg font-semibold">Errors: {{ errors }}</p>
        </div>
        <div class="mb-6 p-4 bg-gray-100 rounded-lg">
          <p class="text-xl leading-relaxed" v-html="highlightedText"></p>
        </div>
        <textarea
          v-model="userInput"
          @input="checkInput"
          :placeholder="!paused ? 'Start typing here...' : ''"
          class="w-full h-64 p-4 border rounded-lg focus:outline-none focus:border-blue-500 text-lg"
          :disabled="!started || time === 0"
        ></textarea>
        <div class="mt-4 flex justify-between">
          <button
            @click="pauseResumeTest"
            class="bg-yellow-500 text-white py-2 px-4 rounded-lg hover:bg-yellow-600"
          >
            {{ paused ? 'Resume' : 'Pause' }}
          </button>
          <button
            @click="resetTest"
            class="bg-red-500 text-white py-2 px-4 rounded-lg hover:bg-red-600"
          >
            Reset
          </button>
        </div>
        <div v-if="time === 0" class="mt-6 text-center">
          <p class="text-2xl font-bold mb-4">Your final typing speed is {{ typingSpeed }} WPM</p>
          <div class="mb-6 space-y-2">
            <p class="text-xl">Total Words: {{ totalWords }}</p>
            <p class="text-xl">Total Errors: {{ errors }}</p>
            <p class="text-xl">Accuracy: {{ accuracy }}%</p>
          </div>
          <button
            @click="resetTest"
            class="w-full bg-green-600 text-white py-3 px-6 rounded-lg hover:bg-green-700 text-xl"
          >
            Restart
          </button>
        </div>
        <div v-else class="mt-6 text-center">
          <p class="text-2xl font-bold">Current Speed: {{ currentSpeed }} WPM</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      sampleTexts: {
        english: [
          'The quick brown fox jumps over the lazy dog. This sentence is often used to test typing speed. It contains all the letters of the alphabet. Typing it quickly and accurately is a good way to measure your typing skills. Keep practicing to improve your speed and accuracy.',
          'Pack my box with five dozen liquor jugs. This sentence is also a pangram, containing every letter of the alphabet. It is shorter than the previous sentence but still effective for testing typing speed and accuracy.',
          'How quickly daft jumping zebras vex. This is another example of a pangram, though it is quite short. It can be useful for quick typing tests and practicing speed with a variety of letters.'
        ],
        indonesian: [
          'Pagi hari yang cerah di sebuah desa kecil. Penduduk desa tersebut hidup dengan damai dan sejahtera. Mereka bekerja sama untuk menciptakan lingkungan yang nyaman dan harmonis. Setiap hari mereka saling membantu dalam berbagai kegiatan.',
          'Bulan purnama bersinar terang di langit malam. Cahaya bulan menyinari setiap sudut desa. Anak-anak bermain di halaman rumah mereka, sementara orang tua duduk bercengkerama sambil menikmati teh hangat.',
          'Di tengah hutan yang lebat, terdengar suara burung berkicau. Hutan tersebut menjadi rumah bagi berbagai jenis hewan dan tumbuhan. Keindahan alam yang memukau membuat siapa saja yang datang merasa tenang dan damai.'
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
      durations: [30, 60, 120]
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
      // Clear userInput and errors when pausing to prevent placeholder access
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
    }
  }
}
</script>

<style>
body {
  font-family: 'Inter', sans-serif;
}
</style>
