<template>
    <v-dialog
        v-model="dialog"
        width="50rem" >

        <template v-slot:activator="{ on, attrs }">
            <v-btn icon dark v-bind="attrs" v-on="on">
                <v-icon>mdi-information-outline</v-icon>
            </v-btn>
        </template>

        <v-card class="mx-auto">
            <v-card-title>
                Request maker
            </v-card-title>
            <v-card-text>
                <p>Developed and maintained by Maxime Moreillon</p>
                <p>Version v{{version}}</p>
            </v-card-text>

        </v-card>
    </v-dialog>
    
</template>

<script>

import { version } from '@/../package.json'

export default {
    name: 'About',
    data() {
        return {
            version,
            dialog: false,
        }
    },


}
</script>

<style scoped>
.about {
    text-align: center;
}
</style>
