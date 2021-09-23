<template>
  <div class="columns">
    <a class="github-fork-ribbon left-top" href="https://github.com/andi2201/tess-ocr-fe" data-ribbon="Fork me on GitHub" title="Fork me on GitHub">Fork me on GitHub</a>
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
/*!
 * "Fork me on GitHub" CSS ribbon v0.2.3 | MIT License
 * https://github.com/simonwhitaker/github-fork-ribbon-css
*/

.github-fork-ribbon {
  width: 12.1em;
  height: 12.1em;
  position: absolute;
  overflow: hidden;
  top: 0;
  right: 0;
  z-index: 9999;
  pointer-events: none;
  font-size: 13px;
  text-decoration: none;
  text-indent: -999999px;
}

.github-fork-ribbon.fixed {
  position: fixed;
}

.github-fork-ribbon:hover, .github-fork-ribbon:active {
  background-color: rgba(0, 0, 0, 0.0);
}

.github-fork-ribbon:before, .github-fork-ribbon:after {
  /* The right and left classes determine the side we attach our banner to */
  position: absolute;
  display: block;
  width: 15.38em;
  height: 1.54em;

  top: 3.23em;
  right: -3.23em;

  -webkit-box-sizing: content-box;
  -moz-box-sizing: content-box;
  box-sizing: content-box;

  -webkit-transform: rotate(45deg);
  -moz-transform: rotate(45deg);
  -ms-transform: rotate(45deg);
  -o-transform: rotate(45deg);
  transform: rotate(45deg);
}

.github-fork-ribbon:before {
  content: "";

  /* Add a bit of padding to give some substance outside the "stitching" */
  padding: .38em 0;

  /* Set the base colour */
  background-color: hsl(153, 53%, 53%);

  /* Set a gradient: transparent black at the top to almost-transparent black at the bottom */
  background-image: -webkit-gradient(linear, left top, left bottom, from(rgba(0, 0, 0, 0)), to(rgba(0, 0, 0, 0.15)));
  background-image: -webkit-linear-gradient(top, rgba(0, 0, 0, 0), rgba(0, 0, 0, 0.15));
  background-image: -moz-linear-gradient(top, rgba(0, 0, 0, 0), rgba(0, 0, 0, 0.15));
  background-image: -ms-linear-gradient(top, rgba(0, 0, 0, 0), rgba(0, 0, 0, 0.15));
  background-image: -o-linear-gradient(top, rgba(0, 0, 0, 0), rgba(0, 0, 0, 0.15));
  background-image: linear-gradient(to bottom, rgba(0, 0, 0, 0), rgba(0, 0, 0, 0.15));

  /* Add a drop shadow */
  -webkit-box-shadow: 0 .15em .23em 0 rgba(0, 0, 0, 0.5);
  -moz-box-shadow: 0 .15em .23em 0 rgba(0, 0, 0, 0.5);
  box-shadow: 0 .15em .23em 0 rgba(0, 0, 0, 0.5);

  pointer-events: auto;
}

.github-fork-ribbon:after {
  /* Set the text from the data-ribbon attribute */
  content: attr(data-ribbon);

  /* Set the text properties */
  color: #fff;
  font: 700 1em "Helvetica Neue", Helvetica, Arial, sans-serif;
  line-height: 1.54em;
  text-decoration: none;
  text-shadow: 0 -.08em rgba(0, 0, 0, 0.5);
  text-align: center;
  text-indent: 0;

  /* Set the layout properties */
  padding: .15em 0;
  margin: .15em 0;

  /* Add "stitching" effect */
  border-width: .08em 0;
  border-style: dotted;
  border-color: #fff;
  border-color: rgba(255, 255, 255, 0.7);
}

.github-fork-ribbon.left-top, .github-fork-ribbon.left-bottom {
  right: auto;
  left: 0;
}

.github-fork-ribbon.left-bottom, .github-fork-ribbon.right-bottom {
  top: auto;
  bottom: 0;
}

.github-fork-ribbon.left-top:before, .github-fork-ribbon.left-top:after, .github-fork-ribbon.left-bottom:before, .github-fork-ribbon.left-bottom:after {
  right: auto;
  left: -3.23em;
}

.github-fork-ribbon.left-bottom:before, .github-fork-ribbon.left-bottom:after, .github-fork-ribbon.right-bottom:before, .github-fork-ribbon.right-bottom:after {
  top: auto;
  bottom: 3.23em;
}

.github-fork-ribbon.left-top:before, .github-fork-ribbon.left-top:after, .github-fork-ribbon.right-bottom:before, .github-fork-ribbon.right-bottom:after {
  -webkit-transform: rotate(-45deg);
  -moz-transform: rotate(-45deg);
  -ms-transform: rotate(-45deg);
  -o-transform: rotate(-45deg);
  transform: rotate(-45deg);
}
</style>
  