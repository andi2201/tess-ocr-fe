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

    <div v-if="jobs.length > 0">
      <b-table detailed detail-key="fileName" :data="jobs" striped hoverable>
        <b-table-column field="fileName" label="File" v-slot="props">
          {{ props.row.fileName }}
        </b-table-column>

        <b-table-column field="progressValue" label="Progress" v-slot="props">
          <b-progress
            :type="
              props.row.progressStatus === 'done' ? 'is-success' : 'is-warning'
            "
            :value="props.row.progressValue"
            size="is-medium"
            show-value
          >
            {{ props.row.progressStatus }}
          </b-progress>
        </b-table-column>

        <template #detail="props">
          {{ props.row.result ? props.row.result : '' }}
        </template>
      </b-table>
    </div>
  </section>
</template>

<script>
import { createWorker } from 'tesseract.js'

export default {
  data() {
    return {
      lang: null,
      files: [],
      jobs: [],
    }
  },
  computed: {},
  methods: {
    async inputFile(files) {
      // files that are not in the jobs array:
      const todoFiles = files.filter(
        (file) => !this.jobs.find((job) => job.fileName === file.name)
      )
      todoFiles.forEach((file) => {
        this.jobs.push({ fileName: file.name, file: file })
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
          job.progressStatus = 'done'
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
