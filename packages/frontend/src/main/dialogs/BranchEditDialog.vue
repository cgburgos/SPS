<template>
  <v-card v-if="editableBranch && editableBranch.name !== 'main'" :loading="loading">
    <v-toolbar color="primary" dark flat>
      <v-app-bar-nav-icon style="pointer-events: none">
        <v-icon>mdi-pencil</v-icon>
      </v-app-bar-nav-icon>
      <v-toolbar-title>Edit Branch</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn icon @click="$emit('close')"><v-icon>mdi-close</v-icon></v-btn>
    </v-toolbar>
    <v-alert v-show="error" dismissible type="error">
      {{ error }}
    </v-alert>
    <v-form ref="form" v-model="valid" lazy-validation @submit.prevent="updateBranch">
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
          required
        ></v-text-field>
        <v-text-field
          v-model="origin"
          label="Origin"
          :rules="originRules"
          required
        ></v-text-field> -->
        <v-text-field
          v-model="editableBranchName"
          label="Name"
          :rules="nameRules"
          required
        ></v-text-field>
        <p v-if="valid" class="caption">
          <b>Branch code:</b>
          {{ ` ${specialty}/${origin}/${editableBranchName} ` }}
        </p>
        <p class="caption">
          {{ resText }}
        </p>
        <!-- <v-textarea
          v-model="editableBranch.description"
          rows="2"
          label="Description"
        ></v-textarea> -->
      </v-card-text>
    </v-form>
    <v-card-actions>
      <v-btn text color="error" @click="showDeleteDialog = true">
        <v-icon small>mdi-delete</v-icon>
        <span class="ml-2">Delete</span>
      </v-btn>
      <v-spacer></v-spacer>
      <v-btn text @click="$emit('close')">Cancel</v-btn>
      <v-btn color="primary" :disabled="!valid" @click="updateBranch()">Save</v-btn>
    </v-card-actions>
    <v-dialog v-model="showDeleteDialog" max-width="500">
      <v-card>
        <v-toolbar color="error" dark flat>
          <v-app-bar-nav-icon style="pointer-events: none">
            <v-icon>mdi-pencil</v-icon>
          </v-app-bar-nav-icon>
          <v-toolbar-title>Delete Branch</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-btn icon @click="showDeleteDialog = false">
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </v-toolbar>
        <v-card-text class="mt-4">
          You cannot undo this action. The branch
          <code>{{ initialBranch.name }}</code>
          will be permanently deleted. To confirm, type its name below:
          <v-text-field
            v-model="branchNameConfirmation"
            label="Confirm branch name"
            required
          ></v-text-field>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn text @click="showDeleteDialog = false">Cancel</v-btn>
          <v-btn
            color="error"
            text
            :disabled="branchNameConfirmation !== initialBranch.name"
            @click="deleteBranch()"
          >
            Delete
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-card>
  <v-card v-else>
    <v-card-text>You cannot edit the main branch.</v-card-text>
  </v-card>
</template>
<script>
import { gql } from '@apollo/client/core'
import isNull from 'lodash/isNull'
import isUndefined from 'lodash/isUndefined'
import clone from 'lodash/clone'
import { StreamEvents } from '@/main/lib/core/helpers/eventHubHelper'
import {
  validSpecialtiesArray,
  validOriginArray,
  reservedBranchNamesArray,
  restrictionText
} from '../utils/streamDataManager'

export default {
  props: {
    stream: {
      type: Object,
      default: () => null
    }
  },
  data() {
    return {
      resText: restrictionText,
      dialog: false,
      // Cloning to prevent mutation of this.stream.branch
      editableBranch: clone(this.stream.branch),
      initialBranch: clone(this.stream.branch),
      editableBranchName: clone(this.stream.branch).name.split('/')[2],
      initialBranchName: clone(this.stream.branch).name.split('/')[2],
      branchNameConfirmation: null,
      validSpecialties: validSpecialtiesArray,
      validOrigin: validOriginArray,
      valid: true,
      loading: false,
      showDeleteDialog: false,
      specialty: clone(this.stream.branch).name.split('/')[0],
      origin: clone(this.stream.branch).name.split('/')[1],
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
          (v && reservedBranchNamesArray.findIndex((e) => e === v) === -1) ||
          'This is a reserved branch name',
        (v) => (v && v.length <= 100) || 'Name must be less than 100 characters',
        (v) => (v && v.length >= 3) || 'Name must be at least 3 characters'
      ],
      specialtyRules: [
        (v) => !!v || 'Specialty needs a name!',
        (v) =>
          (v && validSpecialtiesArray.findIndex((e) => e === v) !== -1) ||
          `The specialty has to be one of the following: ${validSpecialtiesArray.join(
            ' | '
          )}`
      ],
      originRules: [
        (v) => !!v || 'Origin needs a name!',
        (v) =>
          (v && validOriginArray.findIndex((e) => e === v) !== -1) ||
          `The origin has to be one of the following: ${validOriginArray.join(' | ')}`
      ],
      isEdit: false,
      pendingDelete: false,
      allBranchNames: [],
      error: null
    }
  },
  apollo: {
    allBranchNames: {
      query: gql`
        query branchNames($id: String!) {
          stream(id: $id) {
            id
            branches {
              items {
                id
                name
              }
            }
          }
        }
      `,
      variables() {
        return {
          id: this.$route.params.streamId
        }
      },
      update(data) {
        return data.stream.branches.items
          .filter((b) => b.name !== this.stream.branch.name)
          .map((b) => b.name)
      },
      skip() {
        return isNull(this.initialBranch) || isUndefined(this.initialBranch)
      }
    }
  },
  methods: {
    async deleteBranch() {
      this.loading = true
      this.error = null
      this.$mixpanel.track('Branch Action', { type: 'action', name: 'delete' })

      const streamId = this.$route.params.streamId
      const branchId = this.initialBranch.id

      try {
        const res = await this.$apollo.mutate({
          mutation: gql`
            mutation branchDelete($params: BranchDeleteInput!) {
              branchDelete(branch: $params)
            }
          `,
          variables: {
            params: {
              streamId,
              id: branchId
            }
          }
        })
        if (!res.data.branchDelete) throw new Error('Something went wrong!')
      } catch (err) {
        this.$eventHub.$emit('notification', { text: err.message })
        return
      }

      this.loading = false
      this.showDelete = false
      this.showDeleteDialog = false
      this.editableBranch = null
      this.$eventHub.$emit('notification', { text: 'Branch deleted' })
      this.$router.push(`/streams/` + this.$route.params.streamId)
      this.$emit('close')
      this.$eventHub.$emit(StreamEvents.RefetchBranches)
    },
    async updateBranch() {
      if (!this.$refs.form.validate()) return
      try {
        const newBranchCode =
          `${this.specialty}/${this.origin}/${this.editableBranchName}`.toLowerCase()
        if (this.allBranchNames.indexOf(newBranchCode) !== -1)
          throw new Error('Branch already exists. Please choose a different name.')

        this.loading = true
        this.$mixpanel.track('Branch Action', { type: 'action', name: 'update' })

        const branchId = this.editableBranch.id
        const newName = newBranchCode
        const newDescription = this.editableBranch.description

        const res = await this.$apollo.mutate({
          mutation: gql`
            mutation branchUpdate($params: BranchUpdateInput!) {
              branchUpdate(branch: $params)
            }
          `,
          variables: {
            params: {
              streamId: this.$route.params.streamId,
              id: branchId,
              name: newName,
              description: newDescription
            }
          }
        })

        if (!res.data.branchUpdate) throw new Error('Something went wrong!')
      } catch (err) {
        this.$eventHub.$emit('notification', { text: err.message })
      }

      this.loading = false
      this.$eventHub.$emit(StreamEvents.RefetchBranches)
      this.$eventHub.$emit('notification', {
        text: 'Branch updated',
        action: {
          name: 'View',
          to:
            `/streams/` +
            this.$route.params.streamId +
            `/branches/` +
            this.editableBranch.name
        }
      })
      this.$router.push(
        `/streams/` +
          this.$route.params.streamId +
          `/branches/` +
          this.editableBranch.name
      )
      this.$emit('close')
    }
  }
}
</script>
