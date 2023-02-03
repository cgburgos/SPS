<template>
  <div class="latest-blogposts">
    <v-card
      rounded="lg"
      style="overflow: hidden"
      class="transparent elevation-0 pb-4"
      flat
    >
      <v-toolbar
        class="mt-3 elevation-0"
        style="border: 1px solid #d2dad3; border-radius: 10px"
        rounded="lg"
        dense
      >
        <v-toolbar-title class="body-2 font-weight-bold">Tutorials</v-toolbar-title>
        <v-spacer />
        <v-app-bar-nav-icon href="https://speckle.systems/tutorials" target="_blank">
          <v-icon small>mdi-open-in-new</v-icon>
        </v-app-bar-nav-icon>
      </v-toolbar>
    </v-card>
  </div>
</template>
<script>
import GhostContentAPI from '@tryghost/content-api'

export default {
  data() {
    return {
      posts: []
    }
  },
  mounted() {
    this.api = new GhostContentAPI({
      url: 'https://speckle.systems',
      key: 'bf4ca76b9606d0c13b0edf5dc1',
      version: 'v3'
    })

    this.api.posts
      .browse({
        filter: 'tags:[tutorials,blog]',
        limit: 7
      })
      .then((posts) => {
        this.posts = posts
      })
      .catch((err) => {
        this.$eventHub.$emit('notification', {
          text: err.message
        })
      })
  }
}
</script>
<style lang="scss" scoped>
.latest-blogposts {
  margin-top: 27px;
  width: 240px;
}
</style>
