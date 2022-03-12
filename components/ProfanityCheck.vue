<template>
  <div>
    <label :for="'file-upload'" class="hover:bg-gray-50 mb-8 max-w-lg cursor-pointer flex justify-center px-6 pt-5 pb-6 border-2 border-gray-300 transition duration-300 border-dashed rounded-md h-48 items-center" :class="{ 'bg-gray-200' : isActive}" @dragover="setActive" @dragleave="cancelActive" @drop="fileAdded">
      <div class="space-y-1 text-center">
        <svg xmlns="http://www.w3.org/2000/svg" class="mx-auto h-12 w-12 mb-2 text-gray-400" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
          <path stroke-linecap="round" stroke-linejoin="round" d="M19 11a7 7 0 01-7 7m0 0a7 7 0 01-7-7m7 7v4m0 0H8m4 0h4m-4-8a3 3 0 01-3-3V5a3 3 0 116 0v6a3 3 0 01-3 3z" />
        </svg>
        <div class="flex flex-col text-sm text-gray-600">
          <div  class="relative cursor-pointe rounded-md font-medium text-indigo-600 hover:text-indigo-500 focus-within:outline-none focus-within:ring-2 focus-within:ring-offset-2 focus-within:ring-indigo-500">
            <span>Select a local audio file</span>
            <input :id="'file-upload'" :accept="accept" :multiple="false" @change="fileAdded" :name="'file-upload'" type="file" class="sr-only" />
          </div>
                  </div>
        <p class="text-xs text-gray-500">
          Any mp3/wav under 10MB that you want to analyze
        </p>
      </div>
    </label>

    <vue-plyr v-if="selectedFile.url">
      <audio controls crossorigin playsinline>
        <source
          :src="selectedFile.url"
          :type="selectedFile.type"
        />
      </audio>
    </vue-plyr>

    <div class="border p-2">
      <span v-for="(word, key) in transcription" :key="key">
        {{ word.punctuated_word + ' ' }} 
      </span>
    </div>
  </div>
</template>

<script>
import Robodog from '@uppy/robodog'
var BadWords = require('bad-words')
var BadWordsSDK = new BadWords()
import '@uppy/robodog/dist/robodog.css'
export default {
  data() {
    return {
      accept: 'audio/*',
      isActive: false,
      selectedFile: {
        url: null,
        file: null
      },
      transcription: null
    }
  },
  mounted() {
    console.log(BadWordsSDK.isProfane("fuck"))
    console.log(BadWordsSDK.isProfane("sweet"))
  },
  methods: {
    cancelHandlers (e) {
      e.preventDefault()
      e.stopPropagation()
    },
    setActive (e) {
      this.isActive = true
      this.cancelHandlers(e)
    },
    cancelActive (e) {
      this.isActive = false
      this.cancelHandlers(e)
    },
    fileAdded (e) {
        this.isActive = false
        this.cancelHandlers(e)
        const wasDropped = e.dataTransfer
        const files = wasDropped ? e.dataTransfer.files : e.target.files
        if (wasDropped && !this.allowMultiple && files.length > 1) alert("Multiple Files not allowed")
        if (wasDropped && this.requiresTypeCheck) {
          for (var i = 0; i < files.length; i++) {
          }
        } else {
          for (var i = 0; i < files.length; i++) {
            console.log(files[i])
            this.selectedFile.url = URL.createObjectURL(files[i])
            this.selectedFile.file = files[i]
            this.uploadFile()
          }
        }
    },
    uploadFile() {
      const resultPromise = Robodog.upload([this.selectedFile.file], {
        params: {
          auth: { key: '9cd7fdad0615482a94a6580f62aaa981' },
          template_id: '095f1d2b30ba41b5ace5c26e81920d3a',
        },
        waitForEncoding: true
      })
      resultPromise.then((e) => {
        this.selectedFile.url = e.transloadit[0].uploads[0].ssl_url
        this.uploadToDeepGram()
      })
    },
    async uploadToDeepGram() {
      try {
        const response = await this.$axios.post('https://api.deepgram.com/v1/listen?language=en-GB&model=general&punctuate=true', {
          url: this.selectedFile.url
        }, {
          headers: {
            'Authorization': 'Token 5b42b2fe53552e3f665de2d17834b6729aa8de4d'
          }
        })
        console.log(response.data.results.channels[0].alternatives[0].words)
        this.transcription = response.data.results.channels[0].alternatives[0].words
      } catch (e) {
        console.log(e)
      } finally {

      }
    }
  }
}
</script>

<style>

</style>