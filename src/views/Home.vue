<template>

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
                <v-col>
                  <v-select label="Method" :items="methods" item-text="text" item-value="value"
                    v-model="request.method" />
                </v-col>
              </v-row>
              <v-row>
                <v-col>
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
                    <v-tab>
                      Headers
                    </v-tab>
                    <v-tab v-if="['post', 'put', 'patch'].includes(request.method)">
                      Body (JSON)
                    </v-tab>
                  </v-tabs>
                </v-toolbar>
                <v-divider />
                <v-card-text>
                  <v-tabs-items v-model="tab">
                    <!-- Header tabs -->
                    <v-tab-item>
                      <v-row>
                        <v-col>
                          <h3>Headers</h3>
                        </v-col>
                        <v-spacer />
                        <v-col cols="auto">
                          <v-btn small @click="add_header()">
                            <v-icon>mdi-plus</v-icon>
                            <span>Add header</span>
                          </v-btn>
                        </v-col>
                      </v-row>
                      <table v-if="request.headers.length">
                        <tr v-for="(item, index) in request.headers" v-bind:key="`header_item_${index}`">
                          <td>
                            <v-text-field v-model="request.headers[index].key" placeholder="Key" />
                          </td>
                          <td>:</td>
                          <td>
                            <v-text-field v-model="request.headers[index].value" placeholder="Value" />
                          </td>
                          <td>
                            <v-btn icon @click="delete_header(index)">
                              <v-icon>mdi-delete</v-icon>
                            </v-btn>
                          </td>
                        </tr>
                      </table>

                      <div class="" v-else>
                        No headers
                      </div>
                    </v-tab-item>

                    <!-- Body tab -->
                    <v-tab-item>
                      <v-row>
                        <v-col>
                          <h3>Body (JSON)</h3>
                        </v-col>

                        <v-spacer />

                        <v-col cols="auto">
                          <v-btn small @click="add_body_item()">
                            <v-icon>mdi-plus</v-icon>
                            <span>Add body item</span>
                          </v-btn>
                        </v-col>
                      </v-row>

                      <template v-if="request.body.length">
                        <div class=""> { </div>
                        <table>
                          <tr v-for="(item, index) in request.body" :key="`body_item_${index}`">
                            <td></td>
                            <td>"</td>
                            <td>
                              <v-text-field v-model="request.body[index].key" placeholder="Key" />
                            </td>
                            <td>" : "</td>
                            <td>
                              <v-text-field v-model="request.body[index].value" placeholder="Value" />
                            </td>
                            <td>
                              <span>"</span>
                              <span v-if="index < request.body.length -1">,</span>
                            </td>
                            <td>
                              <v-btn icon @click="delete_body_item(index)">
                                <v-icon>mdi-delete</v-icon>
                              </v-btn>
                            </td>

                          </tr>
                        </table>
                        <div class=""> } </div>
                      </template>

                      <div v-else>
                        Empty body
                      </div>
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
                    <v-icon>mdi-send</v-icon>
                    <span>Send</span>
                  </v-btn>
                </v-col>
                <v-col cols="auto">
                  <v-btn @click="cancel_request()" :disabled="!processing">
                    <v-icon>mdi-close</v-icon>
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
</template>

<script>
import Response from '../components/Response.vue'
import RequestHistoryDialog from '../components/RequestHistoryDialog.vue'

export default {
  name: 'Home',
  components: {
    Response,
    RequestHistoryDialog
  },
  data(){
    return {

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
        {text: 'GET', value: 'get'},
        {text: 'POST', value: 'post'},
        {text: 'DELETE', value: 'delete'},
        {text: 'PUT', value: 'put'},
        {text: 'PATCH', value: 'patch'},
      ],

      request: {
        url: 'http://192.168.1.2:8080/items',
        method: 'get',
        body: [],
        headers: [],
      },

      request_history: [],


      response: null,

    }
  },
  mounted(){
    this.load_history()
  },
  methods: {
    add_request_to_history(){
      const last_item = this.request_history[this.request_history.length-1]
      if (JSON.stringify(this.request) === JSON.stringify(last_item)) return
      this.request_history.push({...this.request})
      if (this.request_history.length > 10) this.request_history.shift()
      this.save_history()
    },
    save_history(){
      const history_stringified = JSON.stringify(this.request_history)
      localStorage.request_history = history_stringified
    },
    load_history(){
      const history_stringified = localStorage.request_history
      if (!history_stringified) return
      try {
        this.request_history = JSON.parse(history_stringified)
        const last_item = this.request_history[this.request_history.length -1 ]
        if (last_item)
        this.request = {...last_item}
      } catch (error) {
        console.warn(error)
      }
    },
    send_request(){

      this.add_request_to_history()

      this.processing = true
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
      } = parsed_url

      const url = `${protocol}//${hostname}:${port}${pathname}`

      const body = this.request.body.reduce( (acc, item) => {
         acc[item.key] = item.value
         return acc
      }, {})

      const headers = this.request.headers.reduce( (acc, header) => ({ ...acc, [header.key]: header.value }), {})

      const axios_options = {
        method: this.request.method,
        url,
        data: body,
        headers,
        signal: this.abortController.signal,
      }


      this.axios(axios_options)
        .then(response => {
          this.response = response
        })
        .catch(error => {

          if(error.response) {
            this.response = error.response
          }
          else {
            console.error(error)
            this.response = error
          }

        })
        .finally( () => { this.processing = false })
    },
    cancel_request(){
      if(!this.processing) return
      this.abortController.abort()
    },
    add_body_item(){
      this.request.body.push({key: '', value: ''})
    },
    delete_body_item(index){
      this.request.body.splice(index, 1)
    },
    add_header(){
      this.request.headers.push({key: '', value: ''})
    },
    delete_header(index){
      this.request.headers.splice(index, 1)
    },
  },
  computed: {
    
    url_valid(){
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

<style scoped>


table {
  width: 100%;
  border-collapse: collapse;
}

input, select, button {
  padding: 0.25em;
}

td, th {
  padding: 0.25em;
  white-space: nowrap;
}

table input,
table select,
table button {
  width: 100%;
}

</style>
