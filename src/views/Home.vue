<template>
  <div class="home">

    <header>
      <h1>Request maker</h1>
      <p>A homemade (and very rudimentary) version of Postman, developped by Maxime MOREILLON</p>
    </header>


    <div class="columns">
      <form class="request_form" @submit.prevent="send_request()">
        <h2>Request</h2>
        <h3>Method</h3>
        <select class="method_select" v-model="request.method">
          <option
            v-for="(method, index) in methods"
            v-bind:key="`method_${index}`"
            :value="method">
            {{method}}
          </option>
        </select>

        <h3>URL</h3>

        <table class="url">
          <tr>
            <th class="protocol">Protocol</th>
            <th class="separator"></th>
            <th class="hostname">Hotname / IP</th>
            <th class="separator"></th>
            <th class="port">Port</th>
            <th class="route">Route</th>
          </tr>

          <tr>

            <td>
              <select class="" v-model="request.protocol">
                <option
                  v-for="(protocol, index) in protocols"
                  v-bind:key="`protocol_${index}`"
                  :value="protocol">
                  {{protocol}}
                </option>
              </select>
            </td>

            <td>://</td>

            <td>
              <input type="text" v-model="request.hostname" placeholder="192.168.1.2">
            </td>

            <td>:</td>

            <td>
              <input type="text" v-model="request.port" placeholder="8080">
            </td>

            <td>
              <input type="text" v-model="request.route" placeholder="/users">
            </td>


          </tr>

        </table>

        <h3>Headers</h3>

        <p>
          <button
            type="button"
            @click="add_header()">
            Add header
          </button>
        </p>

        <table v-if="request.headers.length > 0">
          <tr>
            <th>Key</th>
            <th></th>
            <th>Value</th>
            <th>Delete</th>
          </tr>
          <tr
            v-for="(item, index) in request.headers"
            v-bind:key="`header_item_${index}`">
            <td>
              <input
                type="text"
                 v-model="request.headers[index].key">
            </td>
            <td>:</td>
            <td>
              <input
                type="text"
                v-model="request.headers[index].value">
            </td>
            <td>
              <button
                type="button"
                @click="delete_header(index)">
                Delete
              </button>
            </td>
          </tr>
        </table>

        <div class="" v-else>
          No headers
        </div>

        <h3>Body (JSON)</h3>

        <p>
          <button
            type="button"
            @click="add_body_item()">
            Add item
          </button>
        </p>

        <template v-if="request.body.length > 0">
          <div class=""> { </div>
          <table >
            <!--
            <tr>
              <th></th>
              <th>Key</th>
              <th></th>
              <th>Value</th>
              <th>Delete</th>
            </tr>
            -->
            <tr
              v-for="(item, index) in request.body"
              v-bind:key="`body_item_${index}`">
              <td>"</td>
              <td>
                <input
                  type="text"
                  placeholder="Key"
                  v-model="request.body[index].key">
              </td>
              <td>" : "</td>
              <td>
                <input
                  type="text"
                  placeholder="Value"
                  v-model="request.body[index].value">
              </td>
              <td>
                <span>"</span>
                <span v-if="index < request.body.length -1">,</span>
              </td>
              <td>
                <button
                  type="button"
                  @click="delete_body_item(index)">
                  Delete
                </button>
              </td>

            </tr>
          </table>
          <div class=""> } </div>
        </template>

        <div class="" v-else>
          Empty body
        </div>

        <h3>Send</h3>
        <input type="submit">

      </form>

      <div class="">
        <h2>Response</h2>

        <template v-if="response.status.code">
          <h3>Status</h3>

          <p class="status">
            {{response.status.code}} {{response.status.text}}
          </p>
        </template>

        <template v-if="response.body">
          <h3>Body</h3>
          <p class="body">
            {{response.body}}
          </p>
        </template>
      </div>
    </div>


  </div>
</template>

<script>
export default {
  name: 'Home',
  data(){
    return {

      methods: [
        'get',
        'post',
        'delete',
        'put',
        'patch',
      ],

      protocols: [
        'http',
        'https'
      ],

      request: {
        method: 'get',
        protocol: 'http',
        hostname: '192.168.1.2',
        port: 8576,
        route: '',
        body: [],
        headers: [],
      },


      // Response
      response: {

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

      const url = `${this.request.protocol}://${this.request.hostname}:${this.request.port}${this.request.route}`

      let body = {}
      this.request.body.forEach((item) => { body[item.key] = item.value })

      let headers = {}
      this.request.headers.forEach((item) => { headers[item.key] = item.value })


      this.axios({
        method: this.request.method,
        url: url,
        data: body,
        headers: headers,
      })
      .then(response => {
        this.response.status.code = response.status
        this.response.status.text = response.statusText
        this.response.body = response.data
      })
      .catch(error => {

        if(error.response) {
          this.response.status.code = error.response.status
          this.response.status.text = error.response.statusText
          this.response.body = error.response.data
        }
        else {
          console.log(error)
          this.response.body = `Network error`
        }

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
  }

}
</script>

<style scoped>
header {
  text-align: center;
}
.columns {
  display: flex;
}

.columns > *:not(:last-child) {
  margin-right: 1em;
}

.columns > * {
  border: 1px solid #dddddd;
  border-radius: 0.5em;
  padding: 1em;
  flex-grow: 1;
  flex-basis: 0;
  flex-shrink: 1;
}

.columns {

}
.method_select, .method_select option {
  text-transform: uppercase;
}

.url {
  //table-layout: fixed;
}

.protocol {
  width: 10%;
}

.separator {
  width: 4%;
}

.hostname {
  width: 20%;
}

.port {
  width: 10%;
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
}


table input,
table select,
table button {
  width: 100%;

}

</style>
