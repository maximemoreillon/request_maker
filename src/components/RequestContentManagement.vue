<template>
    <div>
        <v-row>
            <v-col>
                <h3>Body</h3>
            </v-col>
        
            <v-spacer />
        
        
        </v-row>
        <v-row align="baseline">
            <v-col>
                <v-select :items="body_types" v-model="content.type" label="Content-Type" />
            </v-col>
            <v-col cols="auto">
                <v-btn small @click="add_body_item()">
                    <v-icon left>mdi-plus</v-icon>
                    <span>Add item</span>
                </v-btn>
            </v-col>
        </v-row>
        
        <!-- Request content type is JSON -->
        <template v-if="content.type === 'json'">
            <v-textarea label="JSON" :rows="1" auto-grow :rules="jsonRules" placeholder='{"message": "Hello World!"}'
                v-model="content.json" />
        
        </template>
        
        <!-- If request body content type is multipart/form-data -->
        <template v-if="content.type === 'multipart'">
        
            <template v-if="content.formData.length">
                <v-row align="baseline" v-for="(item, index) in content.formData" :key="`body_item_${index}`">
                    <v-col>
                        <v-text-field v-model="content.formData[index].key" label="Field" />
                    </v-col>
                    <v-col>
                        <v-select label="Field type" :items="formData_field_types" item-text="text" item-value="value"
                            v-model="content.formData[index].type" />
                    </v-col>
                    <v-col>
                        <v-text-field v-if="content.formData[index].type === 'string'"
                            v-model="content.formData[index].value" placeholder="Value" />
                        <v-file-input v-else-if="content.formData[index].type === 'file'"
                            v-model="content.formData[index].value" />
                    </v-col>
                    <v-col cols="1">
                        <v-btn icon @click="delete_body_item(index)">
                            <v-icon>mdi-delete</v-icon>
                        </v-btn>
                    </v-col>
                </v-row>
            </template>
        
            <div v-else>
                Empty body
            </div>
        </template>
    </div>
</template>

<script>
export default {
    name: 'RequestContentManagement',
    props: {
        value: Object
    },
    data(){
        return {
            body_types: [
                { text: 'application/json', value: 'json' },
                { text: 'Multipart/form-data', value: 'multipart' },
            ],
            formData_field_types: [
                { text: 'String', value: 'string' },
                { text: 'File', value: 'file' },
            ],
            jsonRules: [
                v => {
                    try {
                        JSON.parse(v)
                        return true
                    } catch { return 'JSON is invalid' }

                },
            ],
        }
    },
    methods: {
        add_body_item() {
            if (this.content.type === 'json') this.content.json.push({ key: '', value: '' })
            else if (this.content.type === 'multipart') this.content.formData.push({ key: '', value: null, type: 'string' })
        },
        delete_body_item(index) {
            if (this.content.type === 'json') this.content.json.splice(index, 1)
            else if (this.content.type === 'multipart') this.content.formData.splice(index, 1)
        },
    },
    computed: {
        content: {
            get() {
                return this.value
            },
            set(newVal) {
                this.$emit('input', newVal)
            }
        }
    }
}

</script>