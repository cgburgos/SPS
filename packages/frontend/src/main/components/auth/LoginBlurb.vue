<template>
  <v-card class="elevation-0 transparent pa-5 pb-0">
    <v-card-text class="text-h3 text-sm-h4 text-md-h3 primary--text">
      <span class="primary--text">
        <b>
          <!-- display: inline -->
          <a
            class="text-decoration-none"
            href="https://www.unstudio.com"
            target="_blank"
            >UNSpeckle</a
          >
        </b>
      </span>
      <!-- display: inline -->
      <span class="font-weight-light">, at the core of our design!</span>
    </v-card-text>
    <div>
      <v-card-text class="text--text text-h6 font-weight-regular">
        UNSpeckle is the platform that we developed to connect your projects, teams and
        data in a
        <b>single app</b>
        . With a large array of tools and features that we hope that makes your designs
        better, your projects more efficient and your
        <b>life easier</b>
        .
      </v-card-text>
      <v-card-text class="text--text">
        Currently in {{ ServerPack.displayVersion }} with HoPI v{{
          ServerPack.HoPIVersion
        }}
      </v-card-text>
    </div>
  </v-card>
</template>
<script>
import ServerPackage from '../../../../../../package.json'

export default {
  name: 'LoginBlurb',
  data() {
    return {
      ServerPack: ServerPackage
    }
  }
}
</script>
