<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
      <div class="d-flex align-center">
        <v-img
          alt="SNOMED International Logo"
          class="shrink mr-2"
          contain
          src="https://www.snomed.org/SNOMED/media/SNOMED/other/brand-mark.png"
          transition="scale-transition"
          width="200"
          height="60"
        />
        <h1>SNOMED CT IPS Sub-Ontology Coding Guide</h1>
      </div>
      <v-spacer></v-spacer>
      <v-btn color="purple lighten-1">
          <vue-excel-xlsx
            :data="bindings"
            :columns="columns"
            :file-type="'xlsx'"
            :file-name="'ecl-export'"
            :sheet-name="'sheet1'"
            >
            <v-icon right dark class="mr-4">mdi-cloud-download</v-icon>EXPORT BINDINGS
          </vue-excel-xlsx>
      </v-btn>
      <v-menu offset-y>
        <template v-slot:activator="{ on, attrs }">
          <v-btn
            color="purple lighten-1"
            dark
            v-bind="attrs"
            v-on="on"
            class="ml-4"
          >
            <v-icon dark>
              mdi-information
            </v-icon>
          </v-btn>
        </template>
        <v-list>
          <v-list-item-group>
            <v-list-item>
              <v-list-item-icon>
                <v-icon>mdi-github</v-icon>
              </v-list-item-icon>
              <v-list-item-title @click="goToGithub()">
                Source code on GitHub
              </v-list-item-title>
            </v-list-item>
          </v-list-item-group>
          <v-subheader>Bound to SNOMED CT Release:</v-subheader>
          <v-list-item>
            <v-list-item-title>
              Edition
            </v-list-item-title>
            <v-list-item-subtitle>
              Int. Edition
            </v-list-item-subtitle>
          </v-list-item>
          <v-list-item>
            <v-list-item-title>
              Version
            </v-list-item-title>
            <v-list-item-subtitle>
              {{codeSystemVersionDisplay}}
            </v-list-item-subtitle>
          </v-list-item>
        </v-list>
      </v-menu>
    </v-app-bar>

    <v-main>
      <MainTabs :sections="sections"/>
    </v-main>
  </v-app>
</template>

<script>
import MainTabs from './components/MainTabs.vue';

import VueExcelXlsx from "vue-excel-xlsx";
import Vue from "vue";

Vue.use(VueExcelXlsx);

export default {
  name: 'App',

  components: {
    MainTabs,
  },
  mounted() {
    this.bindings = [];
    for (const section in this.sections) {
      for (const binding in this.sections[section].bindings) {
        this.bindings.push({ section: this.sections[section].title, title: this.sections[section].bindings[binding].title, ecl: this.sections[section].bindings[binding].ecl.replace(/\s\s+/g, ' ') })
      }
    }
    Vue.prototype.$snowstormBase = 'https://snowstorm.ihtsdotools.org/snowstorm/snomed-ct';
    Vue.prototype.$codeSystemVersion = '2022-01-31';
    // Vue.prototype.$snowstormBase = 'https://iaa.snomed.tools/snowstorm/snomed-ct';
    // Vue.prototype.$codeSystemVersion = 'SNOMEDCT-IPS-TEST2';
    Vue.prototype.$snowstormBranch = `MAIN/${this.$codeSystemVersion}`;
    this.codeSystemVersionDisplay = this.$codeSystemVersion;
  },
  methods: {
    goToGithub() {
      window.open('https://github.com/alopezo/coding-demonstrator', '_blank');
    }
  },
  data: () => ({
    bindings: [],
    columns : [
                    {
                        label: "Section",
                        field: "section",
                    },
                    {
                        label: "Title",
                        field: "title",
                    },
                    {
                        label: "ECL",
                        field: "ecl",
                    }
                ],
    codeSystemVersionDisplay: '',
    sections: {
        'IPS-ALLERGIES': {
          title: 'Allergies and Intolerances',
          note: 'This section documents the relevant allergies or intolerances (conditions) for a patient, describing the kind of reaction (e.g. rash, anaphylaxis,..); preferably the agents that cause it; and optionally the criticality and the certainty of the allergy. At a minimum, it should list currently active and any relevant historical allergies and adverse reactions. If no information about allergies is available, or if no allergies are known this should be clearly documented in the section.',
          bindings: {
            'IPS-ALLERGIES-AllergyintolerancesubstanceconditionGPS' : {
              title: 'Allergy intolerance substance condition (GPS) - IPS',
              type: 'autocomplete',
              ecl: `(< 762766007 | Edible substance (substance)| OR
              < 425620007 | Metal (substance)| OR 
              < 410942007 |Drug or medicament (substance)| OR 
              < 373873005 |Pharmaceutical / biologic product (product)|) AND
              (^816080008 |International Patient Summary|)`,
              value: '',
              note: 'IPS Allergy intolerance substance condition GPS value set. This value set includes the codes from the SNOMED International Global Patient Set (GPS) subset of SNOMED CT that are included in: all descendants of 762766007 | Edible substance (substance)|; all descendants of 406455002 | Allergen class (substance) |; all descendants of 425620007 | Metal (substance)|; all descendants of 373873005 | Pharmaceutical / biologic product (product)|; all descendants of 410942007 |Drug or medicament (substance)|. The current value set contains concepts from the September 2019 release of the GPS, which is based on the July 2019 SNOMED CT International Edition release.'
            }
          }
        },
        'IPS-PROBLEMS': {
          title: 'Problem List',
          note: 'The IPS problem section lists and describes clinical problems or conditions currently being monitored for the patient.',
          bindings: {
            'IPS-PROBLEMS-CoreProblemListFindingSituationEventGpsUvIps' : {
              title: 'CORE Problem List Finding/Situation/Event (GPS) - IPS',
              type: 'autocomplete',
              ecl: `(< 404684003 |Clinical finding (finding)| OR 
              < 272379006 |Event (event)| OR 
              (< 243796009 |Situation with explicit context (situation)| : [0..0] 363589002 |Associated procedure (attribute)| = *)) AND 
              (^816080008 |International Patient Summary|)`,
              value: '',
              note: 'This value set is a special subset for the International Patient Summary of the concepts in the CORE Problem List Subset of SNOMED CT® which are also contained in the SNOMED CT Global Patient Set (GPS). This value set includes the concepts which are in the Clinical finding, Situation with explicit context and Event hierarchies, but excludes the concepts from the Procedure hierarchy, as they are expected to be represented separately in the History of Procedures Section. '
            },
            'IPS-PROBLEMS-CoreProblemListFindingSituationEventAllSNOMED' : {
              title: 'CORE Problem List Finding/Situation/Event (All SNOMED)',
              type: 'autocompleterefset',
              ecl: `(< 404684003 |Clinical finding (finding)| OR 
              < 272379006 |Event (event)| OR 
              (< 243796009 |Situation with explicit context (situation)| : [0..0] 363589002 |Associated procedure (attribute)| = *))`,
              refset: '816080008 |International Patient Summary|',
              value: '',
              note: 'This value set is a special subset for the International Patient Summary of the concepts in the CORE Problem List Subset of SNOMED CT® which are also contained in the SNOMED CT Global Patient Set (GPS). This value set includes the concepts which are in the Clinical finding, Situation with explicit context and Event hierarchies, but excludes the concepts from the Procedure hierarchy, as they are expected to be represented separately in the History of Procedures Section.'
            }
          }
        }
      }
  }),
};
</script>