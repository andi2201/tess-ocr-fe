<template>
  <section>
    <b-field>
      <b-upload v-model="files" multiple drag-drop @input="inputFile">
        <section class="section">
          <div class="content has-text-centered">
            <p>
              <b-icon icon="upload" size="is-large"> </b-icon>
            </p>
            <p>Drop your files here or click to upload</p>
          </div>
        </section>
      </b-upload>
    </b-field>
    <div v-for="job in jobs" :key="job.fileName">
      <b-field label="Progress">
        <div class="content has-text-centered is-3">
          <b-progress :value="job.progressValue" size="is-medium" show-value>
            {{ job.progressStatus }}
          </b-progress>
        </div>
      </b-field>
      <b-field label="Message">
        {{ job.result ? job.result : '' }}
      </b-field>
    </div>

    <!-- <div class="tags">
      <span
        v-for="(file, index) in dropFiles"
        :key="index"
        class="tag is-primary"
      >
        {{ file.name }}
        <button
          class="delete is-small"
          type="button"
          @click="deleteDropFile(index)"
        ></button>
      </span>
    </div> -->
  </section>
</template>

<script>
// import Tesseract from 'tesseract.js'
import { createWorker } from 'tesseract.js'

export default {
  data() {
    return {
      lang: null,
      files: [],
      jobs: [],
      // socket: { readyState: 3 },
    }
  },
  // mounted() {
  //   this.initSocket()
  // },
  computed: {},
  methods: {
    async inputFile(files) {
      // console.log(this.socket)
      // this.socket.send(file)
      files.forEach((file) => {
        if (!this.jobs.find((job) => job.fileName === file.name)) {
          this.jobs.push({ fileName: file.name, file: file })
        }
      })

      for (const job of this.jobs) {
        if (!job.progressValue) {
          const worker = createWorker({
            logger: (m) => {
              job.progressStatus = m.status
              job.progressValue = m.progress * 100
              console.log(m)
              this.jobs.splice()
            },
          })

          await worker.load()
          await worker.loadLanguage(this.lang ?? 'deu')
          await worker.initialize(this.lang ?? 'deu')

          const {
            data: { text },
          } = await worker.recognize(job.file)
          job.result = text
          this.jobs.splice()
        }
      }
    },
    // initSocket() {
    //   this.reconnectDisabled = true

    //   // dev
    //   // this.socket = new WebSocket(
    //   //   'ws://9c67b16a-f5a5-4f35-ad9e-c6b8708cd52b.pub.instances.scw.cloud:8080'
    //   // )

    //   // prod
    //   this.socket = new WebSocket(
    //     'wss://9c67b16a-f5a5-4f35-ad9e-c6b8708cd52b.pub.instances.scw.cloud:8080'
    //   )

    //   this.socket.onopen = () => {
    //     this.reconnectDisabled = false
    //   }

    //   this.socket.onerror = (err) => {
    //     this.reconnectDisabled = false
    //   }

    //   this.socket.onclose = () => {
    //     this.reconnectDisabled = false
    //   }

    //   this.socket.onmessage = ({ data }) => {
    //     const serverMessage = JSON.parse(data)
    //     console.log(serverMessage)

    //     if (serverMessage.text) {
    //       this.text = serverMessage.text
    //     } else {
    //       this.progressStatus = serverMessage.status
    //       this.progressValue = serverMessage.progress * 100
    //     }
    //   }
    // },
    deleteDropFile(index) {
      this.files.splice(index, 1)
    },
  },
}
</script>
