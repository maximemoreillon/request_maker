<template>
    <v-dialog v-model="dialog" width="500">
        <template v-slot:activator="{ on, attrs }">
            <v-btn v-bind="attrs" v-on="on">
                History
            </v-btn>
        </template>
        <v-card>
            <v-card-title>
                Request history
            </v-card-title>
            <v-card-text>
                <v-list>
                    <v-list-item @click="$emit('loadRequest', item)" v-for="(item, index) in history_reversed"
                        :key="index">
                        {{item.method.toUpperCase()}} {{item.url}}
                    </v-list-item>
                </v-list>
            </v-card-text>
            <v-card-actions>
                <v-spacer />
                <v-btn text @click="dialog = false">close</v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>
</template>

<script>
export default {
    name: 'RequestHistory',
    props: {
        history: Array,
    },
    data(){
        return {
            dialog: false,
        }
    },
    computed: {
        history_reversed(){
            return this.history.slice().reverse()
        }
    }


}
</script>

<style scoped>
</style>