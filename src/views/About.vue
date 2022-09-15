<template>
  <v-card max-width="40rem" class="mx-auto">
    <v-card-title>
      Request maker
    </v-card-title>
    <v-card-text>
      <p>Developed and maintained by Maxime Moreillon</p>
      <p>Version v{{version}}</p>
    </v-card-text>
    
  </v-card>
</template>

<script>

import {version} from '@/../package.json'

export default {
  name: 'About',
  data(){
    return {
      version
    }
  },


}
</script>

<style scoped>
.about {
  text-align: center;
}
</style>
