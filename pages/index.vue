<template>
  <div class="columns">
    <div class="column"></div>
    <div class="column is-three-fifths is-centered">
      <h1 class="content has-text-centered" style="margin-top: 2em">
        OCR in your Browser powered by
        <a href="https://tesseract.projectnaptha.com/">tesseract.js</a>
      </h1>
      <div class="cent" style="margin-top: 2em; margin-bottom: 2em">
        <b-field label="Language">
          <b-autocomplete
            clearable
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

      <div class="cent">
        <b-upload v-model="files" multiple drag-drop @input="inputFile">
          <section class="section">
            <div class="content has-text-centered">
              <p>
                <b-icon icon="upload" size="is-large"> </b-icon>
              </p>
              <p>Drop your files here or click to start OCR</p>
            </div>
          </section>
        </b-upload>
      </div>
      <div v-if="jobs.length > 0">
        <b-table detailed detail-key="fileName" :opened-detailed="openedDetailed" :data="jobs" striped hoverable>
          <b-table-column field="fileName" label="File" v-slot="props">
            {{ props.row.fileName }}
          </b-table-column>

          <b-table-column
            width="300"
            field="progressValue"
            label="Progress"
            v-slot="props"
          >
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
            <span v-if="props.row.result">
              <div class="has-text-right">
                <b-tooltip label="copy text">
                  <a @click="copyToClipboard(props.row.result)"
                    ><b-icon icon="content-copy"></b-icon></a
                ></b-tooltip>
                <br />
              </div>
              {{ props.row.result }}
            </span>
            <span v-else> please wait </span>
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
      openedDetailed: [],
    }
  },
  computed: {
    // openedDetailed(){
    //   return jobs.map((job) => job.fileName)
    // },
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
    copyToClipboard(text) {
      navigator.clipboard.writeText(text)
      this.$buefy.toast.open({
        duration: 2000,
        message: `Ok, cool! 👍`,
        position: 'is-bottom',
      })
    },
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
          try {
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
            this.openedDetailed.push(job.fileName)
          } catch {
            job.result = '🤷‍♀️'
            job.progressStatus = 'error'
          } finally {
            this.jobs.splice()
          }
        }
      }
    },
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
}
</style>
  