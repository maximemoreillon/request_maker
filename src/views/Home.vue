<template>

    <v-row
      class="debug"
      align-content="stretch">
      <!-- Request column -->
      <v-col>
        <v-sheet
          class="pa-5"
          elevation="1"
          rounded>

          <h2>Request</h2>

          <v-form
            @submit.prevent="send_request()"
            ref="form"
            v-model="valid"
            lazy-validation>

            <v-select
              label="Method"
              :items="methods"
              v-model="request.method"/>

            <v-text-field
              label="URL"
              v-model="request_url"
              placeholder="http://192.169.1.2:8080/users"
              required
              :rules="urlRules"/>


            <v-row class="mt-2">
              <v-col>
                <h3>Headers</h3>
              </v-col>

              <v-spacer></v-spacer>

              <v-col>
                <v-btn
                  small
                  @click="add_header()">
                  <v-icon>mdi-plus</v-icon>
                  <span>Add header</span>
                </v-btn>
              </v-col>
            </v-row>

            <table v-if="request.headers.length > 0">
              <tr
                v-for="(item, index) in request.headers"
                v-bind:key="`header_item_${index}`">
                <td>
                  <v-text-field
                    v-model="request.headers[index].key"
                    placeholder="Key"/>
                </td>
                <td>:</td>
                <td>
                  <v-text-field
                    v-model="request.headers[index].value"
                    placeholder="Value"/>
                </td>
                <td>
                  <v-btn
                    icon
                    @click="delete_header(index)">
                    <v-icon>mdi-close</v-icon>
                  </v-btn>
                </td>
              </tr>
            </table>

            <div class="" v-else>
              No headers
            </div>

            <template v-if="['post', 'put', 'patch'].includes(request.method)">

              <v-row class="mt-3">
                <v-col>
                  <h3>Body (JSON)</h3>
                </v-col>

                <v-spacer></v-spacer>

                <v-col>
                  <v-btn
                    small
                    @click="add_body_item()">
                    <v-icon>mdi-close</v-icon>
                    <span>Add body item</span>
                  </v-btn>
                </v-col>
              </v-row>

              <template v-if="request.body.length > 0">
                <div class=""> { </div>
                <table >
                  <tr
                    v-for="(item, index) in request.body"
                    v-bind:key="`body_item_${index}`">
                    <td></td>
                    <td>"</td>
                    <td>
                      <v-text-field
                        v-model="request.body[index].key"
                        placeholder="Key"/>
                    </td>
                    <td>" : "</td>
                    <td>
                      <v-text-field
                        v-model="request.body[index].value"
                        placeholder="Value"/>
                    </td>
                    <td>
                      <span>"</span>
                      <span v-if="index < request.body.length -1">,</span>
                    </td>
                    <td>
                      <v-btn
                        icon
                        @click="delete_body_item(index)">
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

            <v-row class="mt-6">
              <v-col
                class="text-center"
                cols="12" >
                <v-btn
                  type="submit">
                  <v-icon>mdi-send</v-icon>
                  <span>Send request</span>
                </v-btn>
              </v-col>
            </v-row>

          </v-form>
        </v-sheet>
      </v-col>
      <!-- Response column -->
      <v-col>
        <v-sheet
          elevation="1"
          rounded
          class="pa-5">
          <h2>Response</h2>

          <p v-if="response.loading">
            Processing request...
          </p>

          <template v-if="response.status.code">
            <h3>Status</h3>
            <p :class="{error_message: response.status.code.toString().charAt(0) !== '2', success_message: true}">
              {{response.status.code}} {{response.status.text}}
            </p>
          </template>

          <template v-if="response.body">
            <h3>Body</h3>

            <p
              class=""
              v-if="response.headers['content-type'].includes('text/html')"
              v-html="response.body"/>

            <p v-else>
              {{response.body}}
            </p>

          </template>

          <template v-if="response.error">
            <h3>Error</h3>
            <p class="error_message">
              {{response.error}}
            </p>
          </template>
        </v-sheet>
      </v-col>

    </v-row>

</template>

<script>
export default {
  name: 'Home',
  data(){
    return {

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
        'get',
        'post',
        'delete',
        'put',
        'patch',
      ],

      protocols: [
        'http:',
        'https:'
      ],

      request: {
        method: 'post',
        protocol: 'http:',
        hostname: '192.168.1.2',
        port: 8576,
        route: '/users',
        body: [],
        headers: [],
        url: 'http://192.168.1.2:8080/users',
      },


      // Response
      response: {

        error: null,

        loading: false,

        headers: null,

        status: {
          code: null,
          text: null,
        },

        body: null,
      },




    }
  },
  methods: {
    send_request(){

      this.response.loading = true

      this.response.error = null,
      this.response.status.code = null
      this.response.status.text = null
      this.response.body = null
      this.response.headers = null

      const url = `${this.request.protocol}//${this.request.hostname}:${this.request.port}${this.request.route}`


      const body = this.request.body.reduce( (acc, item) => {
         acc[item.key] = item.value
         return acc
      }, {})

      const headers = this.request.headers.reduce( (acc, header) => {
         acc[header.key] = header.value
         return acc
      }, {})


      this.axios({
        method: this.request.method,
        url,
        data: body,
        headers,
      })
      .then(response => {

        this.response.status.code = response.status
        this.response.status.text = response.statusText
        this.response.body = response.data
        this.$set(this.response, 'headers', response.headers)
      })
      .catch(error => {

        if(error.response) {
          this.$set(this.response, 'headers', error.response.headers)
          this.response.status.code = error.response.status
          this.response.status.text = error.response.statusText
          this.response.body = error.response.data
        }
        else {
          console.log(error)
          this.response.error = `Network error`
        }

      })
      .finally( () => {
        this.response.loading = false
      })
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
    request_url: {

      get () {
        return this.request.url
      },
      set (url_string) {

        let url

        try {url = new URL(url_string)}
        catch {return}

        const {
          hostname,
          port,
          protocol,
          pathname,
          search,
        } = url



        this.request = {
          ...this.request,
          hostname,
          port,
          protocol,
          route: pathname,
          search,
        }

      }
    }
  }

}
</script>

<style scoped>
.debug {
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
