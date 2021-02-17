<template>
  <v-app>
    <v-main>
      <nuxt />
    </v-main>
  </v-app>
</template>

<script lang="ts">
import { Vue, Component } from 'nuxt-property-decorator'

@Component({
  components: {},
})
export default class search extends Vue {
  drawer: boolean = false
  drawer2: boolean = false

  df: any = {}
}
</script>
