<template>
    <v-dialog
      v-model="dialog"
      scrollable
      max-width="800px"
    >
        <template v-slot:activator="{ on, attrs }">
            <v-btn
            color="success ml-4"
            dark
            v-bind="attrs"
            v-on="on"
            >
            Binding
            </v-btn>
        </template>

        <v-card>
            <v-card-title class="text-h5 grey lighten-2">
            {{ binding.title }}
            </v-card-title>
             <v-card-subtitle class="text-h6 grey lighten-2" v-if="binding.fieldTitle">
            {{ binding.fieldTitle }}
            </v-card-subtitle>

            <v-card-text style="height: 800px;">
                <h4 class="my-5">ECL</h4>
                <v-btn
                    icon
                    color="primary"
                    style="float: right;"
                    @click="copy"
                >
                    <v-icon>mdi-content-copy</v-icon>
                </v-btn>
                <pre class="mx-4">{{ binding.ecl }}</pre>
                <h4 class="my-5">API Call</h4>
                <v-btn
                    icon
                    color="primary"
                    style="float: right;"
                    @click="openUrl"
                >
                    <v-icon>mdi-open-in-new</v-icon>
                </v-btn>
                <pre class="mx-4 text-small">{{ queryString.replace(/\s\s+/g, ' ').substring(0, 200) + '...' }}</pre>
                <h4 class="my-5">MEMBERS  ({{ total }})</h4>
                <v-list dense>
                    <v-list-item-group
                        v-model="selectedItem"
                        color="primary"
                    >
                        <v-list-item
                        v-for="(item, i) in items"
                        :key="i"
                        >
                            <v-list-item-content>
                                <v-list-item-title v-text="item.fsn.term"></v-list-item-title>
                                <v-list-item-subtitle>SCTID: {{ item.id }}</v-list-item-subtitle>
                            </v-list-item-content>
                        </v-list-item>
                        <v-list-item v-if="total > items.length && !loading">
                            <v-list-item-content>
                                <v-list-item-subtitle class="text-center" @click="getData()">
                                    {{ items.length }} of {{ total }} Load more...
                                    </v-list-item-subtitle>
                            </v-list-item-content>
                        </v-list-item>
                        <v-list-item v-if="total == items.length && !loading">
                            <v-list-item-content>
                                <v-list-item-subtitle class="text-center">All results loaded ({{ total }})</v-list-item-subtitle>
                            </v-list-item-content>
                        </v-list-item>
                        <v-list-item v-if="loading">
                            <v-list-item-content>
                                <v-progress-linear
                                indeterminate
                                color="cyan"
                                ></v-progress-linear>
                            </v-list-item-content>
                        </v-list-item>
                    </v-list-item-group>
                </v-list>
            </v-card-text>
            <v-divider></v-divider>
        </v-card>
    </v-dialog>
</template>
<style scoped>
pre {
    white-space: pre-wrap;       /* Since CSS 2.1 */
}
</style>
<script>
  import axios from "axios";

    export default {
        data: () => ({
            loading: false,
            items: [],
            total: 0,
            selectedItem: 0,
            dialog: false,
            queryString: ''
        }),
        props: {
            binding: {
                type: Object
            }
        },
        mounted() {
            this.getData();
        },
        methods: {
            copy() {
                navigator.clipboard.writeText(this.binding.ecl.replace(/\s\s+/g, ' '));
            },
            openUrl() {
                window.open(this.queryString.replace(/\s\s+/g, ' '), '_blank').focus();
            },
            getData() {
                this.loading = true;
                var base = this.binding.base || this.$snowstormBase
                var branch = this.binding.branch || this.$snowstormBranch
                this.queryString = `${base}/${branch}/concepts?activeFilter=true&
                            termActive=true&language=en&offset=0&limit=50&ecl=${encodeURIComponent(this.binding.ecl)}`
                axios
                    .get(this.queryString)
                    .then(response => {
                        // this.items = response.data.items.map( e => e.fsn.term );
                        this.items = this.items.concat(response.data.items);
                        this.total = response.data.total;
                        this.loading = false;
                    })
            }
        }
    }
</script>
