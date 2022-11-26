<template>
    <div>
        <v-row>
            <v-col>
                <h3>Headers</h3>
            </v-col>
            <v-spacer />
            <v-col cols="auto">
                <v-btn small @click="add_header()">
                    <v-icon left>mdi-plus</v-icon>
                    <span>Add header</span>
                </v-btn>
            </v-col>
        </v-row>
        <table v-if="headers.length">
            <tr v-for="(item, index) in headers" :key="`header_item_${index}`">
                <td>
                    <v-text-field v-model="headers[index].key" placeholder="Key" />
                </td>
                <td>:</td>
                <td>
                    <v-text-field v-model="headers[index].value" placeholder="Value" />
                </td>
                <td>
                    <v-btn icon @click="delete_header(index)">
                        <v-icon>mdi-delete</v-icon>
                    </v-btn>
                </td>
            </tr>
        </table>
        
        <div class="" v-else>
            No headers
        </div>
    </div>
</template>

<script>
export default {
    name: 'RequestHeadersManagement',
    props: {
        value: Array
    },
    methods: {
        add_header() {
            this.headers.push({ key: '', value: '' })
        },
        delete_header(index) {
            this.headers.splice(index, 1)
        },
    },
    computed: {
        headers: {
            get(){
                return this.value
            },
            set(newVal) {
                console.log(newVal)
                this.$emit('input', newVal)
            }
        }
    }
}

</script>

<style scoped>

table {
    width: 100%;
}
</style>