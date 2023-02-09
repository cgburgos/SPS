<template>
  <v-chip
    v-tooltip="`Source Application: ${applicationName ? applicationName : 'unknown'}`"
    small
    class="ma-1 caption white--text no-hover"
    :color="color"
  >
    {{ shortName }}
  </v-chip>
</template>
<script>
// APP COLOURS
import { BadgeColours } from '@/plugins/colorPalette'
export default {
  props: {
    applicationName: {
      type: String,
      default: '?'
    }
  },
  data: () => ({
    apps: BadgeColours
  }),
  computed: {
    // adding new colors?
    // this can help: https://codepen.io/teocomi/pen/vYxvREG?editors=1010
    color() {
      if (!this.applicationName) return 'grey'

      const appname = this.applicationName.toLowerCase()

      for (const app of this.apps) {
        if (appname.includes(app.name)) return app.color
      }
      return 'grey'
    },
    shortName() {
      if (!this.applicationName) return '?'

      const appname = this.applicationName.toLowerCase()

      for (const app of this.apps) {
        if (appname.includes(app.name)) return app.short
      }
      return appname
    }
  }
}
</script>
