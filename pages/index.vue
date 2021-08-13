<template>
  <div class="columns">
    <div class="column"></div>
    <div class="column is-three-fifths is-centered">
      <div class="cent">
        <b-upload v-model="files" multiple drag-drop @input="inputFile">
          <section class="section">
            <div class="content has-text-centered">
              <p>
                <b-icon icon="upload" size="is-large"> </b-icon>
              </p>
              <p>Drop your files here or click to start <strong>OCR</strong></p>
            </div>
          </section>
        </b-upload>
      </div>
      <div>
        <b-field label="select a language">
          <b-autocomplete
            v-model="selectedLanguage"
            placeholder="e.g. German"
            open-on-focus
            :data="filteredDataObj"
            field="language"
            @select="languageSelected"
          >
          </b-autocomplete>
        </b-field>

      </div>
      

      <div v-if="jobs.length > 0">
        <b-table detailed detail-key="fileName" :data="jobs" striped hoverable>
          <b-table-column field="fileName" label="File" v-slot="props">
            {{ props.row.fileName }}
          </b-table-column>

          <b-table-column field="progressValue" label="Progress" v-slot="props">
            <b-progress
              :type="
                props.row.progressStatus === 'done'
                  ? 'is-success'
                  : 'is-warning'
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
    </div>
    <div class="column"></div>
  </div>
</template>

<script>
import { createWorker } from 'tesseract.js'

export default {
  data() {
    return {
      selectedLanguage: 'English',
      selectedLanguageCode: 'eng',
      files: [],
      jobs: [],
    }
  },
  computed: {
    filteredDataObj() {
      return this.languages.filter((option) => {
        return (
          option.language
            .toString()
            .toLowerCase()
            .indexOf(this.selectedLanguage.toLowerCase()) >= 0
        )
      })
    },
    languages() {
      return this.$store.state.languages
    },
  },
  methods: {
    languageSelected(option) {
      if (option) {
        this.selectedLanguage = option.language
        this.selectedLanguageCode = option.code
      }
    },
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
          await worker.loadLanguage(this.selectedLanguageCode ?? 'eng')
          await worker.initialize(this.selectedLanguageCode ?? 'eng')

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
<style scoped>
.cent {
  display: flex;
  justify-content: center;
  margin-top: 4em;
  margin-bottom: 2em;
}
</style>
  