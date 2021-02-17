<template>
  <component :is="element.name || 'span'">
    <!-- date and time -->
    <template
      v-if="
        (element.name == 'tei-date' || element.name == 'tei-time') &&
        element.attributes &&
        element.attributes.when
      "
    >
      <v-tooltip right>
        <template #activator="{ on, attrs }">
          <span
            v-bind="attrs"
            v-on="on"
            v-html="getTextContent(element)"
          ></span>
        </template>
        <span>{{ element.attributes.when }}</span>
      </v-tooltip>
    </template>
    <template v-else-if="element.type == 'text'">
      {{ element.text }}
    </template>
    <template v-else-if="element.name == 'tei-choice'">
      <v-tooltip right>
        <template #activator="{ on, attrs }">
          <span style="color: #ff5722" v-bind="attrs" v-on="on">{{
            getCorr(element)
          }}</span>
        </template>
        <span>{{ getSic(element) }}</span>
      </v-tooltip>
    </template>
    <template
      v-else-if="element.attributes && element.attributes.rend == 'right'"
    >
      <div :class="'rend-' + element.attributes.rend">
        <aaa
          v-for="(e, index) in element.elements"
          :key="index"
          :element="e"
        ></aaa>
      </div>
    </template>
    <template v-else>
      <template
        v-if="
          element.name == 'tei-pb' &&
          element.attributes &&
          element.attributes.corresp
        "
      >
        <v-icon
          class="ma-1"
          color="purple"
          @click="canvas = facs[element.attributes.corresp.replace('#', '')]"
          >mdi-image</v-icon
        >
        <span class="ma-1" style="color: grey"
          ><small>[Page @{{ element.attributes.corresp }}]</small></span
        >
      </template>
      <aaa
        v-for="(e, index) in element.elements"
        :key="index"
        :element="e"
      ></aaa>
    </template>
  </component>
</template>

<script lang="ts">
import { Vue, Component, Prop } from 'nuxt-property-decorator'
import aaa from '~/components/aaa.vue'

@Component({
  components: {
    aaa,
  },
  name: 'aaa',
})
export default class Menu extends Vue {
  @Prop({})
  element!: any

  get canvas(): any {
    return this.$store.getters.canvas
  }

  set canvas(value) {
    this.$store.commit('setCanvas', value)
  }

  get facs(): any {
    return this.$store.getters.getFacs
  }

  set facs(value) {
    this.$store.commit('setFacs', value)
  }

  getTextContent(element: any) {
    let text = ''
    if (element.text) {
      text += element.text.split('\n').join('<br/>').trim()
    }
    if (element.elements) {
      for (let i = 0; i < element.elements.length; i++) {
        text += this.getTextContent(element.elements[i])
      }
    }
    return text.trim()
  }

  getSic(element: any) {
    let text = this.getTextContent(element)
    const elements = element.elements
    for (let i = 0; i < elements.length; i++) {
      const e = elements[i]
      if (e.name === 'tei-sic') {
        text = this.getTextContent(e)
      }
    }
    return text
  }

  getCorr(element: any) {
    const elements = element.elements
    let text = this.getTextContent(element)
    for (let i = 0; i < elements.length; i++) {
      const e = elements[i]
      if (e.name === 'tei-corr') {
        text = this.getTextContent(e)
      }
    }
    return text
  }
}
</script>
<style scoped>
tei-persName {
  background-color: #ffccbc;
}

tei-placeName {
  background-color: #c8e6c9;
}

tei-date {
  background-color: #bbdefb;
}

tei-time {
  background-color: #fff9c4;
}

tei-head {
  margin: 20px;
  font-size: large !important;
  font-weight: bold;
}

tei-del {
  text-decoration: line-through;
}

tei-add {
  color: #4caf50;
  text-decoration: underline;
}

tei-hi {
  font-weight: bold;
  text-decoration: underline;
}

tei-emph {
  text-decoration: underline;
}

.rend-right {
  text-align: right;
}
</style>
