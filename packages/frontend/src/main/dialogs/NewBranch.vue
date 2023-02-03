<template>
  <v-card>
    <v-toolbar color="primary" dark flat>
      <v-app-bar-nav-icon style="pointer-events: none">
        <v-icon>mdi-source-branch</v-icon>
      </v-app-bar-nav-icon>
      <v-toolbar-title>New Branch</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn icon @click="$emit('close')"><v-icon>mdi-close</v-icon></v-btn>
    </v-toolbar>
    <v-alert v-model="showError" dismissible type="error">
      {{ error }}
    </v-alert>
    <v-form ref="form" v-model="valid" lazy-validation @submit.prevent="submit">
      <v-card-text>
        <v-combobox
          v-model="specialty"
          :items="validSpecialties"
          label="Specialty"
          :rules="specialtyRules"
          validate-on-blur
          required
        ></v-combobox>
        <v-combobox
          v-model="origin"
          :items="validOrigin"
          label="Origin"
          :rules="originRules"
          validate-on-blur
          required
        ></v-combobox>
        <!-- <v-text-field
          v-model="specialty"
          label="Specialty"
          :rules="specialtyRules"
          validate-on-blur
          required
          autofocus
        ></v-text-field> -->
        <!-- <v-text-field
          v-model="origin"
          label="Origin"
          :rules="originRules"
          validate-on-blur
          required
        ></v-text-field> -->
        <v-text-field
          v-model="name"
          label="Name"
          :rules="nameRules"
          validate-on-blur
          required
        ></v-text-field>
        <p v-if="valid" class="caption">
          <b>Branch code:</b>
          {{ ` ${specialty}/${origin}/${name} ` }}
        </p>
        <p class="caption">
          {{ resText }}
        </p>
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn color="primary" text :disabled="!valid" type="submit">Save</v-btn>
      </v-card-actions>
    </v-form>
  </v-card>
</template>
<script>
import { gql } from '@apollo/client/core'
import {
  validSpecialtiesArray,
  validOriginArray,
  reservedBranchNamesArray,
  restrictionText
} from '../utils/streamDataManager'

export default {
  data() {
    return {
      resText: restrictionText,
      showError: false,
      error: null,
      streamId: null,
      reservedBranchNames: reservedBranchNamesArray,
      validSpecialties: validSpecialtiesArray,
      validOrigin: validOriginArray,
      valid: false,
      loading: false,
      specialty: '',
      origin: '',
      name: '',
      nameRules: [
        (v) => !!v || 'Need a name too!',
        (v) =>
          !(
            v.startsWith('#') ||
            v.startsWith('/') ||
            v.indexOf('//') !== -1 ||
            v.indexOf(' ') !== -1
          ) || 'Branch names cannot contain "#", "/" or white spaces.',
        (v) =>
          (v && this.reservedBranchNames.findIndex((e) => e === v) === -1) ||
          'This is a reserved branch name',
        (v) => (v && v.length <= 100) || 'Name must be less than 100 characters',
        (v) => (v && v.length >= 3) || 'Name must be at least 3 characters'
      ],
      specialtyRules: [
        (v) => !!v || 'Specialty needs a name!',
        (v) =>
          (v && this.validSpecialties.findIndex((e) => e === v) !== -1) ||
          `The specialty has to be one of the following: ${validSpecialtiesArray.join(
            ' | '
          )}`
      ],
      originRules: [
        (v) => !!v || 'Origin needs a name!',
        (v) =>
          (v && this.validOrigin.findIndex((e) => e === v) !== -1) ||
          `The origin has to be one of the following: ${validOriginArray.join(' | ')}`
      ],
      description: null
    }
  },
  computed: {},
  watch: {
    name(val) {
      this.name = val.toLowerCase()
    }
  },
  methods: {
    async submit() {
      if (!this.$refs.form.validate()) return

      const newBranchCode =
        `${this.specialty}/${this.origin}/${this.name}`.toLowerCase()

      this.loading = true
      this.$mixpanel.track('Branch Action', { type: 'action', name: 'create' })
      try {
        await this.$apollo.mutate({
          mutation: gql`
            mutation branchCreate($params: BranchCreateInput!) {
              branchCreate(branch: $params)
            }
          `,
          variables: {
            params: {
              streamId: this.$route.params.streamId,
              name: newBranchCode,
              description: this.description
            }
          }
        })
        this.showError = false
        this.error = null
        this.loading = false
        this.$emit('refetch-branches')
        this.$emit('close')

        try {
          await this.$router.push(
            `/streams/${this.$route.params.streamId}/branches/${newBranchCode}`
          )
        } catch (routerErr) {
          if (routerErr?.name === 'NavigationDuplicated') {
            // Just created a new branch, while we're on its 404 page
            // Kind of an edge case, so as reloading the page is easier, i'll just do that instead of messing
            // with the Apollo cache
            location.reload()
          } else {
            throw routerErr
          }
        }
      } catch (err) {
        this.showError = true
        if (err.message.includes('branches_streamid_name_unique'))
          this.error = 'A branch with that name already exists.'
        else this.error = err.message
      }
    }
  }
}
</script>
