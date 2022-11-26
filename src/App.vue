<template>
  <AppTemplate :options="options">
    <template v-slot:header>
      <AboutDialog />
    </template>
    <v-container fluid>
    
      <v-row>
        <!-- Request column -->
        <v-col cols="12" md="6">
          <v-card>
    
            <v-toolbar flat>
              <v-row>
                <v-col>
                  <v-toolbar-title>Request</v-toolbar-title>
                </v-col>
                <v-spacer />
                <v-col cols="auto">
                  <RequestHistoryDialog :history="request_history" @loadRequest="request = $event" />
                </v-col>
              </v-row>
    
            </v-toolbar>
    
            <!-- Method and URL -->
            <v-form @submit.prevent="send_request()" ref="form" v-model="valid" lazy-validation>
              <v-card-text>
                <v-row>
                  <v-col cols="3">
                    <v-select label="Method" :items="methods" item-text="text" item-value="value"
                      v-model="request.method" />
                  </v-col>
                  <v-col cols="9">
                    <v-text-field label="URL" v-model="request.url" placeholder="http://192.169.1.2:8080/users" required
                      :rules="urlRules" />
                  </v-col>
                </v-row>
              </v-card-text>
    
              <!-- Headers and Body -->
              <v-card-text>
                <v-card outlined>
                  <v-toolbar flat dense>
                    <v-tabs v-model="tab">
                      <v-tabs-slider />
                      <v-tab>Headers</v-tab>
                      <v-tab v-if="['post', 'put', 'patch'].includes(request.method)">
                        Body
                      </v-tab>
                    </v-tabs>
                  </v-toolbar>
                  <v-divider />
                  <v-card-text>
                    <v-tabs-items v-model="tab">
                      <!-- Header tabs -->
                      <v-tab-item>
                        <RequestHeadersManagement v-model="request.headers" />
                      </v-tab-item>
    
                      <!-- Body tab -->
                      <v-tab-item>
                        <RequestContentManagement v-model="request.content" />
                      </v-tab-item>
                    </v-tabs-items>
                  </v-card-text>
    
                </v-card>
              </v-card-text>
    
              <v-card-text>
    
                <!-- Send button -->
                <v-row>
                  <v-spacer />
                  <v-col cols="auto">
                    <v-btn :loading="processing" :disabled="!url_valid" type="submit">
                      <v-icon left>mdi-send</v-icon>
                      <span>Send</span>
                    </v-btn>
                  </v-col>
                  <v-col cols="auto">
                    <v-btn @click="cancel_request()" :disabled="!processing">
                      <v-icon left>mdi-close</v-icon>
                      <span>Cancel</span>
                    </v-btn>
                  </v-col>
                  <v-spacer />
                </v-row>
    
              </v-card-text>
            </v-form>
    
    
          </v-card>
        </v-col>
        <!-- Response column -->
    
        <v-col cols="12" md="6">
          <Response :response="response" :processing="processing" />
        </v-col>
    
      </v-row>
    </v-container>
  </AppTemplate>
</template>

<script>
import AppTemplate from '@moreillon/vue_application_template_vuetify'
import RequestHeadersManagement from './components/RequestHeadersManagement.vue'
import Response from '@/components/Response.vue'
import RequestHistoryDialog from '@/components/RequestHistoryDialog.vue'
import AboutDialog from '@/components/AboutDialog.vue'
import RequestContentManagement from './components/RequestContentManagement.vue'

export default {
  name: 'Home',
  components: {
    AppTemplate,
    Response,
    RequestHistoryDialog,
    AboutDialog,
    RequestHeadersManagement,
    RequestContentManagement
  },
  data() {
    return {

      options: {
        title: "Request maker",
      },

      processing: false,
      abortController: null,

      valid: false,
      tab: null,

      urlRules: [
        v => !!v || 'URL is required',
        v => {
          try {
            new URL(v)
            return true
          } catch { return 'URL is invalid' }

        },
      ],

      methods: [
        { text: 'GET', value: 'get' },
        { text: 'POST', value: 'post' },
        { text: 'DELETE', value: 'delete' },
        { text: 'PUT', value: 'put' },
        { text: 'PATCH', value: 'patch' },
      ],

      
      request: {
        url: 'http://192.168.1.2:8080/items',
        method: 'get',
        content: {
          type: 'json',
          json: '{}',
          formData: [],
        },
        headers: [],
      },

      request_history: [],


      response: null,

    }
  },
  mounted() {
    this.load_history()
  },
  methods: {
    add_request_to_history() {
      const last_item = this.request_history[this.request_history.length - 1]
      if (JSON.stringify(this.request) === JSON.stringify(last_item)) return
      this.request_history.push({ ...this.request })
      if (this.request_history.length > 10) this.request_history.shift()
      this.save_history()
    },
    save_history() {
      const history_stringified = JSON.stringify(this.request_history)
      localStorage.request_history = history_stringified
    },
    load_history() {
      const history_stringified = localStorage.request_history
      if (!history_stringified) return
      try {
        this.request_history = JSON.parse(history_stringified)
        const last_item = this.request_history[this.request_history.length - 1]
        if (last_item) this.request = { ...last_item }
      } catch (error) {
        console.warn(error)
      }
    },
    send_request() {

      this.add_request_to_history()


      this.response = null
      this.abortController = new AbortController()

      let parsed_url
      try { parsed_url = new URL(this.request.url) }
      catch { return console.error('Invalid URL') }

      const {
        hostname,
        port,
        protocol,
        pathname,
        search
      } = parsed_url

      const url = `${protocol}//${hostname}:${port}${pathname}${search}`

      let data

      if (this.request.content.type === 'json') {
        try {
          data = JSON.parse(this.request.content.json)
        }
        catch (error) {
          alert('JSON is not valid')
          return
        }

      }
      else if (this.request.content.type === 'multipart') {
        const formData = new FormData()
        this.request.content.formData.forEach(({ key, value }) => { formData.append(key, value) })
        data = formData
      }

      const headers = this.request.headers.reduce((acc, header) => ({ ...acc, [header.key]: header.value }), {})

      const axios_options = {
        method: this.request.method,
        url,
        data,
        headers,
        signal: this.abortController.signal,
      }

      this.processing = true


      this.axios(axios_options)
        .then(response => {
          this.response = response
        })
        .catch(error => {

          if (error.response) {
            this.response = error.response
          }
          else {
            console.error(error)
            this.response = error
          }

        })
        .finally(() => { this.processing = false })
    },
    cancel_request() {
      if (!this.processing) return
      this.abortController.abort()
    },

    
  },
  computed: {

    url_valid() {
      try {
        new URL(this.request.url)
        return true
      }
      catch {
        return false
      }
    }
  }

}
</script>