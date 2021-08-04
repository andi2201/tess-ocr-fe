<template>
  <section>
    <b-field v-if="socket.readyState !== 1">
      <b-button :disabled="reconnectDisabled" @click="initSocket">
        Connect {{ socket.readyState }}</b-button
      >
    </b-field>
    <b-field v-else>
      <b-upload v-model="dropFiles" drag-drop @input="inputFile">
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
    <b-field label="Progress">
      <div class="content has-text-centered is-3">
        <b-progress :value="progressValue" size="is-medium" show-value>
          {{ progressStatus }}
        </b-progress>
      </div>
    </b-field>
  <b-field label="Message">
            <b-input v-model="text" type="textarea"></b-input>
        </b-field>


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
export default {
  data() {
    return {
      dropFiles: [],
      socket: { readyState: 3 },
      reconnectDisabled: false,
      progressStatus: '',
      progressValue: 0,
      text: ''
    }
  },
  mounted() {
    this.initSocket()
  },
  computed: {},
  methods: {
    inputFile(file) {
      console.log(this.socket)
      this.socket.send(file)
    },
    initSocket() {
      this.reconnectDisabled = true

      this.socket = new WebSocket('ws://127.0.0.1:8080')

      this.socket.onopen = () => {
        this.reconnectDisabled = false
      }

      this.socket.onerror = (err) => {
        this.reconnectDisabled = false
      }

      this.socket.onclose = () => {
        this.reconnectDisabled = false
      }


      this.socket.onmessage = ({ data }) => {
        const serverMessage = JSON.parse(data)
        console.log(serverMessage)

        if (serverMessage.text) {
          this.text = serverMessage.text
        } else {
          this.progressStatus = serverMessage.status
          this.progressValue = serverMessage.progress * 100
        }
      }
    },
    deleteDropFile(index) {
      this.dropFiles.splice(index, 1)
    },
  },
}
</script>
