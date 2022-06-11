<template>

  <v-container fluid>

    <v-row align="stretch">
      <!-- Request column -->
      <v-col cols="6">
        <v-sheet class="pa-5" elevation="1" rounded>

          <h2>Request</h2>

          <v-form @submit.prevent="send_request()" ref="form" v-model="valid" lazy-validation>

            <v-select class="mt-5" label="Method" :items="methods" item-text="text" item-value="value"
              v-model="request.method" />

            <v-text-field label="URL" v-model="request.url" placeholder="http://192.169.1.2:8080/users" required
              :rules="urlRules" />


            <v-row class="mt-5">
              <v-col>
                <h3>Headers</h3>
              </v-col>

              <v-spacer></v-spacer>

              <v-col>
                <v-btn small @click="add_header()">
                  <v-icon>mdi-plus</v-icon>
                  <span>Add header</span>
                </v-btn>
              </v-col>
            </v-row>

            <table v-if="request.headers.length > 0">
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
                    <v-icon>mdi-close</v-icon>
                  </v-btn>
                </td>
              </tr>
            </table>

            <div class="" v-else>
              No headers
            </div>

            <template v-if="['post', 'put', 'patch'].includes(request.method)">

              <v-row class="mt-5">
                <v-col>
                  <h3>Body (JSON)</h3>
                </v-col>

                <v-spacer></v-spacer>

                <v-col>
                  <v-btn small @click="add_body_item()">
                    <v-icon>mdi-plus</v-icon>
                    <span>Add body item</span>
                  </v-btn>
                </v-col>
              </v-row>

              <template v-if="request.body.length > 0">
                <div class=""> { </div>
                <table>
                  <tr v-for="(item, index) in request.body" v-bind:key="`body_item_${index}`">
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

              <div class="" v-else>
                Empty body
              </div>
            </template>

            <v-row class="mt-8">
              <v-spacer />
              <v-col cols="auto">
                <v-btn :loading="processing" :disabled="processing || !url_valid" type="submit">
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

          </v-form>
        </v-sheet>
      </v-col>
      <!-- Response column -->
      <v-col cols="6">
        <v-sheet elevation="1" rounded class="pa-5" height="100%">

          <h2>Response</h2>

          <p v-if="!processing && !response">
            No response yet
          </p>


          <v-progress-linear class="mt-5" v-if="processing" indeterminate />


          <template v-if="response">
            <template v-if="response.headers">
              <h3>Status</h3>
              <p :class="{error_message: response.status.toString().charAt(0) !== '2', success_message: true}">
                {{response.status}} {{response.statusText}}
              </p>
              <h3>Content-type</h3>
              <p>
                {{response.headers['content-type']}}
              </p>

              <h3>Body</h3>

              <p class="response_body" v-if="response.headers['content-type'].includes('text/html')"
                v-html="response.data" />

              <p class="response_body" v-else-if="response.headers['content-type'].includes('application/json')">
              <pre>{{response_pretty}}</pre>
              </p>

              <p class="response_body" v-else>
                {{ response.data }}
              </p>
            </template>
            <template v-else>
              <p>Request failed</p>
            </template>


          </template>



        </v-sheet>
      </v-col>

    </v-row>

  </v-container>



</template>

<script>
export default {
  name: 'Home',
  data(){
    return {

      processing: false,
      abortController: null,

      valid: false,

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


      response: null,

    }
  },
  mounted(){
    this.load_request()
  },
  methods: {
    save_request(){
      const request_stringified = JSON.stringify(this.request)
      localStorage.request = request_stringified
    },
    load_request(){
      const request_stringified = localStorage.request
      if (!request_stringified) return
      try {
        this.request = JSON.parse(request_stringified)
      } catch (error) {
        console.warn(error)
      }
    },
    send_request(){

      this.save_request()

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
            alert('Request failed, see console for details')
          }

        })
        .finally( () => { this.processing = false })
    },
    cancel_request(){
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
    response_pretty() {
      try {
        return JSON.stringify(JSON.parse(this.response.data), null, 2)
      }
      catch (error) {
        return this.response.data
      }
    },
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
.response_body {
  width: 100%;
  overflow-x: auto;
}
.success_message {
  color: #00c000;
}
.error_message {
  color: #c00000;
}

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
