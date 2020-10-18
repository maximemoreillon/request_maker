<template>
  <div class="home">
    <h1>Request maker</h1>
    <p>A homemade (and very rudimentary) version of Postman, developped by Maxime MOREILLON</p>

    <h2>Request</h2>


    <form class="request_form" @submit.prevent="send_request()">

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
      <table>
        <tr>
          <th>Protocol</th>
          <th></th>
          <th>Hotname / IP</th>
          <th></th>
          <th>Port</th>
          <th>Route</th>
          <th>Send</th>
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
            <input type="text" v-model="request.hostname">
          </td>

          <td>:</td>

          <td>
            <input type="text" v-model="request.port">
          </td>

          <td>
            <input type="text" v-model="request.route">
          </td>

          <td>
            <input type="submit">
          </td>

        </tr>

      </table>










    </form>

    <h3>Body (JSON)</h3>

    <p>
      <button
        type="button"
        @click="add_body_item()">
        Add item
      </button>
    </p>

    <table>
      <tr>
        <th>Key</th>
        <th>Value</th>
        <th>Delete</th>
      </tr>
      <tr
        v-for="(item, index) in request.body"
        v-bind:key="`body_item_${index}`">
        <td>
          <input
            type="text"
             v-model="request.body[index].key">
        </td>
        <td>
          <input
            type="text"
            v-model="request.body[index].value">
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



    <h3>Headers</h3>

    <p>
      <button
        type="button"
        @click="add_header()">
        Add header
      </button>
    </p>

    <table>
      <tr>
        <th>Key</th>
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

    <h2>Response</h2>

    <p class="status">
      Status: {{response.status.code}} {{response.status.text}}
    </p>

    <p class="body">
      {{response.body}}
    </p>




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
        route: '/users',
        body: [{key: 'username', value: 'john'}],
        headers: [{key: 'authorization', value: 'bearer 123'}],
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

.method_select, .method_select option {
  text-transform: uppercase;
}

table {
  width: 100%;
  text-align: left;
  border-collapse: collapse;
}

input, select, button {
  padding: 0.25em;
}

td, th {
  padding: 0.25em;
}
tr:not(:last-child) {
  border-bottom: 1px solid #dddddd;
}

table input,
table select,
table button {
  width: 100%;

}

</style>
