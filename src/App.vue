<template>
  <v-app>
    <v-app-bar
      app
      color="#444"
      dark >

      <v-img
        alt="Logo"
        class="shrink mr-2"
        contain
        src="@/assets/logo.png"
        transition="scale-transition"
        width="40" />

      <v-toolbar-title>API tester</v-toolbar-title>



    </v-app-bar>

    <v-main class="grey lighten-4">
      <v-container
        fluid>
        <router-view />
      </v-container>
    </v-main>

    <v-footer padless>
      <v-row>
        <v-col
          class="text-center"
          cols="12" >
          <span> API tester v{{version}}-</span>
           Maxime Moreillon - {{new Date().getFullYear()}}
        </v-col>
      </v-row>
    </v-footer>


  </v-app>
</template>

<script>
import psjon from '@/../package.json'
export default {
  name: 'App',

  data: () => ({
    version: psjon.version,
  }),
};
</script>

<style scoped>
.footer {
  text-align: center;
}
</style>
