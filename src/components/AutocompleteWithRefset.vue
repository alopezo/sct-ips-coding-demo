<template>
    <v-container>
      <v-autocomplete
          v-model="binding.value"
          :label="binding.title"
          :items="items"
          :loading="loading"
          :search-input.sync="search"
          no-data-text="Type at least 3 characters to search..."
          cache-items
          clearable
          :multiple="binding.isMultiple == true"
          @click:clear="items=[]"
      ></v-autocomplete>
      <v-btn
        v-if="binding.value"
        color="primary"
        @click="mapToRefset()"
      >
        Map to refset
      </v-btn>
      <v-progress-linear v-if="mapping" class="mt-6"
      indeterminate
      color="cyan"
      ></v-progress-linear>
      <v-list flat v-if="maps && maps.length && binding.value">
        <v-subheader>Candidate concepts in target refset: {{ binding.refset }}:</v-subheader>
          <v-list-item
            v-for="(item, i) in maps"
            :key="i"
          >
            <v-list-item-content>
              <v-list-item-title v-text="`${item.conceptId} |${item.fsn.term}|`" class="text-left"></v-list-item-title>
            </v-list-item-content>
          </v-list-item>
      </v-list>
    </v-container>
</template>
<script>
  import axios from "axios";

  export default {
    data: () => ({
        search: null,
        loading: false,
        items: [],
        conceptInRefset: null,
        maps: [],
        timeout: null,
        mapping: false
    }),
    props: {
      binding: {
        type: Object
      }
    },
    watch: {
      search (val) {
        this.items = [];
        this.maps = [];
        if (this.timeout) 
          clearTimeout(this.timeout); 

        this.timeout = setTimeout(() => {
          val && val != this.binding.value && val.length > 2 && this.querySelections(val)
        }, 300); // delay
      },
    },
    methods: {
      querySelections (v) {
        this.loading = true;
        this.items = [];
        this.maps = [];
        var queryString = `${this.$snowstormBase}/${this.$snowstormBranch}/concepts?activeFilter=true&term=${v}&
                            termActive=true&language=en&offset=0&limit=50&ecl=${encodeURIComponent(this.binding.ecl)}`
        axios
          .get(queryString)
          .then(response => {
            this.items = response.data.items.map( x => {
                            x.value = x;
                            x.text = x.fsn.term;
                            return x
                          }); //.map( e => e.fsn.term );
            this.loading = false;
          })
      },
      mapToRefset() {
        this.maps = null;
        this.mapping = true;
        var ancestorsEcl = `(>>${this.binding.value.conceptId}) AND (^${this.binding.refset})`;
        var mapEcl = `(${ancestorsEcl}) MINUS >(${ancestorsEcl})`;
        var queryString = `${this.$snowstormBase}/${this.$snowstormBranch}/concepts?activeFilter=true&
                            termActive=true&language=en&offset=0&limit=50&ecl=${encodeURIComponent(mapEcl)}`
        axios
          .get(queryString)
          .then(response => {
            var matches = response.data.items;
            if (matches.length > 0) {
              if (matches.length == 1) {
                var firstMatch = matches[0];
                if (firstMatch.conceptId == this.binding.value.conceptId) {
                  this.maps = [{conceptId: '', fsn: { term: 'Concept is a member of the refset'}}]
                  this.mapping = false;
                } else {
                  this.maps = matches;
                  this.mapping = false;
                }
              } else {
                this.maps = matches;
                this.mapping = false;
                // var conceptsEcl = matches.map( e => e.conceptId ).join(' OR ');
                // var resolutionEcl = `(${conceptsEcl}) MINUS (>(${conceptsEcl}))`;
                // queryString = `${this.$snowstormBase}/${this.$snowstormBranch}/concepts?activeFilter=true&
                //             termActive=true&language=en&offset=0&limit=50&ecl=${encodeURIComponent(resolutionEcl)}`
                // axios
                //   .get(queryString)
                //   .then(response => {
                //       var results = response.data.items;
                //       this.maps = results;
                //       this.mapping = false;
                //     }
                //   )
              }
            } else {
              console.log(`No map...`);
            }
          })
      }
    }
  }
</script>
