<template>
  <AppTemplate :options="options" />
</template>

<script>
import AppTemplate from '@moreillon/vue_application_template_vuetify'
export default {
  name: 'App',

  components: {
    AppTemplate
  },

  data: () => ({
    options: {
      title: "Request maker",
    },
  }),

};
</script>