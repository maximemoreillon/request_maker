<template>
  <v-app>
    <v-app-bar
      app
      color="#444"
      dark >

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
          <span> API tester -</span>
           Maxime Moreillon - {{new Date().getFullYear()}}
        </v-col>
      </v-row>
    </v-footer>


  </v-app>
</template>

<script>

export default {
  name: 'App',

  data: () => ({
    //
  }),
};
</script>

<style scoped>
.footer {
  text-align: center;
}
</style>
