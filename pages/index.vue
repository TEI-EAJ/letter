<template>
  <div>
    <div>
      <v-navigation-drawer
        v-model="drawer2"
        app
        :temporary="true"
        right
        :width="256 * 2"
      >
        <Metadata></Metadata>
      </v-navigation-drawer>

      <v-app-bar color="primary" dark flat>
        <v-toolbar-title>
          <Title></Title>
        </v-toolbar-title>

        <v-spacer></v-spacer>

        <v-menu offset-y>
          <template #activator="{ on }">
            <v-btn depressed btn color="primary" v-on="on">
              <v-icon class="mr-2">mdi-translate</v-icon>
            </v-btn>
          </template>

          <v-list>
            <v-list-item :to="switchLocalePath('ja')">
              <v-list-item-title>日本語</v-list-item-title>
            </v-list-item>
            <v-list-item :to="switchLocalePath('en')">
              <v-list-item-title>English</v-list-item-title>
            </v-list-item>
          </v-list>
        </v-menu>

        <v-app-bar-nav-icon @click.stop="drawer2 = !drawer2" />
      </v-app-bar>
    </div>

    <template v-if="loading"
      ><div class="pa-10 text-center">
        <v-progress-circular
          indeterminate
          color="primary"
        ></v-progress-circular>
      </div>
    </template>

    <v-container fluid>
      <v-row class="mt-2">
        <v-col cols="12" :sm="manifest ? 6 : 12">
          <v-card
            id="container"
            flat
            outlined
            class="scroll"
            :class="vertical ? 'vertical' : ''"
            :style="`height: ${height * 0.8}px; width: ${
              manifest ? width / 2 : width
            }px;`"
          >
            <div class="pa-4 px-5">
              <aaa :element="element"></aaa>
            </div>
          </v-card>
        </v-col>

        <v-col cols="12" :sm="manifest ? 6 : 12">
          <iframe
            v-show="!loading"
            v-if="manifest"
            :src="baseUrl + `/curation/?manifest=${manifest}&canvas=${canvas}`"
            width="100%"
            :style="`height: ${height * 0.8}px;`"
            allowfullscreen="allowfullscreen"
            frameborder="0"
          >
          </iframe>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import CETEI from 'CETEIcean'
import $ from 'jquery'
import Metadata from '~/components/Metadata3.vue'
import Title from '~/components/Title.vue'
import aaa from '~/components/aaa.vue'

const convert = require('xml-js')

export default {
  components: {
    Metadata,
    Title,
    aaa,
  },
  data() {
    return {
      baseUrl: process.env.BASE_URL || '',
      siteName: process.env.siteName || '',

      github: process.env.github || '',

      loading: false,

      width: window.innerWidth,
      height: window.innerHeight,

      // drawer: false,
      drawer2: false,

      // pos: 1,
      // canvas: '',
      manifest: null,

      // divs: [],
      // ids: {},
      // ids2: {},
      // divs2: [],

      element: {},

      vertical: false,
    }
  },
  head() {
    const title = this.$t(this.siteName)
    return {
      titleTemplate: null,
      title,
    }
  },
  computed: {
    style: {
      get() {
        return this.$store.getters.getStyle
      },
      set(value) {
        this.$store.commit('setStyle', value)
      },
    },

    xml: {
      get() {
        return this.$store.getters.getXml
      },
      set(value) {
        this.$store.commit('setXml', value)
      },
    },

    facs: {
      get() {
        return this.$store.getters.getFacs
      },
      set(value) {
        this.$store.commit('setFacs', value)
      },
    },

    canvas: {
      get() {
        return this.$store.getters.getCanvas
      },
      set(value) {
        this.$store.commit('setCanvas', value)
      },
    },

    id: {
      get() {
        return this.$store.getters.getId
      },
      set(value) {
        this.$store.commit('setId', value)
      },
    },
  },

  mounted() {
    this.loading = true

    window.addEventListener('resize', this.handleResize)

    const query = this.$route.query
    const url = query.u || this.baseUrl + '/data/ex2_sample_finished.xml'
    const CETEIcean = new CETEI()

    // const id = this.$route.query.id

    const self = this
    CETEIcean.getHTML5(url, function (data) {
      console.log('downloaded.')
      self.xml = data

      const dfStr = convert.xml2json(data.outerHTML, {
        compact: false,
        spaces: 4,
      })
      const df = JSON.parse(dfStr)
      self.element = df

      // facs
      const sources = $(data).find('tei-surface')
      const facs = {}
      for (let i = 0; i < sources.length; i++) {
        const source = sources[i]
        facs[$(source).attr('xml:id')] = $(source).attr('source')
      }

      // メイン

      const canvas = facs[Object.keys(facs)[0]]
      self.canvas = canvas

      self.facs = facs
      // マニフェスト
      const manifest = $(data).find('tei-facsimile').attr('source')
      self.manifest = manifest

      self.loading = false

      console.log('processed.')
    })
  },

  methods: {
    handleResize() {
      // resizeのたびにこいつが発火するので、ここでやりたいことをやる
      this.width = window.innerWidth
      this.height = window.innerHeight
    },

    bbb(data) {
      if (!data) {
        return {}
      }
      const dfStr = convert.xml2json(data.outerHTML, {
        compact: false,
        spaces: 4,
      })
      const df = JSON.parse(dfStr)

      return df.elements[0]
    },
  },
}
</script>
<style>
.scroll {
  overflow: auto;
}

.vertical {
  -webkit-writing-mode: vertical-rl;
  -ms-writing-mode: tb-rl;
  writing-mode: vertical-rl;
}
</style>
