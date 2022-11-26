<template>

<div>
    <v-row>
        <v-col>
            <h3>Query parameters</h3>
        </v-col>
        <v-spacer />
        <v-col cols="auto">
            <v-btn small @click="addParameter()">
                <v-icon left>mdi-plus</v-icon>
                <span>Add parameter</span>
            </v-btn>
        </v-col>
    </v-row>
    <table v-if="queryParameters.length">
        <tr v-for="(item, index) in queryParameters" :key="`query_param_${index}`">
            <td>
                <v-text-field v-model="item.key" placeholder="Key" />
            </td>
            <td>:</td>
            <td>
                <v-text-field v-model="queryParameters[index].value" placeholder="Value" />
            </td>
            <td>
                <v-btn icon @click="deleteParameter(index)">
                    <v-icon>mdi-delete</v-icon>
                </v-btn>
            </td>
        </tr>
    </table>

    <div class="" v-else>
        No Query parameters
    </div>
</div>

</template>

<script>
export default {
    name: 'RequstQueryParameters',
    props: {
        value: Array,
    },
    methods: {
        addParameter() {
            this.queryParameters.push({ key: '', value: '' })
        },
        deleteParameter(index) {
            this.queryParameters.splice(index, 1)
        },
    },
    computed: {
        queryParameters: {
            get(){
                return this.value
            },
            set(newVal){
                this.$emit('input', newVal)
            }
        },

    }

}
</script>

<style scoped>
table {
    width: 100%;
}
</style>