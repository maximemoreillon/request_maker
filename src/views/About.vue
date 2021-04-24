<template>
  <div class="about">
    <p>Author: Maxime MOREILLON</p>
    <p>Version: {{version}}</p>

  </div>
</template>

<script>

import pjson from '@/../package.json'

export default {
  name: 'About',
  data(){
    return {
      version: pjson.version
    }
  },


}
</script>

<style scoped>
.about {
  text-align: center;
}
</style>
