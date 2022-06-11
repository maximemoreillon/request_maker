<template>
  <v-card :loading="processing" height="100%">
    <v-toolbar flat>
      <v-row>
        <v-col>
          <v-toolbar-title>Response</v-toolbar-title>
        </v-col>
      </v-row>
    </v-toolbar>

    <v-card-text v-if="!processing && !response">
      No response yet
    </v-card-text>

    <v-card-text v-else-if="!processing && response && response.headers">

      <v-row>
        <v-col cols="4">
          <h3>Status</h3>
        </v-col>
        <v-col cols="auto" :class="{ error_message: response_is_error, success_message: true }">
          {{ response.status }} {{ response.statusText }}
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="4">
          <h3>Content-type</h3>
        </v-col>
        <v-col cols="auto">
          {{ response.headers['content-type'] }}
        </v-col>
      </v-row>
      <v-row>
        <v-col>
          <h3>Body</h3>
        </v-col>
      </v-row>
      <v-row>
        <v-col class="response_body" v-if="content_type.includes('text/html')" v-html="response.data" />
        <v-col class="response_body" v-else-if="content_type.includes('application/json')">
          <pre>{{response_pretty}}</pre>
        </v-col>
        <v-col class="response_body" v-else>
          {{ response.data }}
        </v-col>
      </v-row>





    </v-card-text>

    <v-card-text v-if="response && !response.headers">
      Request failed, see console for details
    </v-card-text>







  </v-card>
</template>

<script>
export default {
  name: 'Response',
  props: {
    processing: Boolean,
    response: null,
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
    content_type(){
      if(!this.response || !this.response.headers) return
      return this.response.headers['content-type']
    },
    response_is_error(){
      const code_first_digit = this.response.status.toString().charAt(0)
      return code_first_digit !== '2' && code_first_digit !== '3'
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
</style>