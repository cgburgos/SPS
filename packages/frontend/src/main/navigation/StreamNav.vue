<template>
  <portal v-if="canRenderNavPortal" to="nav">
    <div v-if="!$loggedIn()" class="px-4 my-2">
      <v-btn small block color="primary" @click="$loginAndSetRedirect()">Sign In</v-btn>
    </div>
    <v-list
      v-if="stream"
      :key="`super-unclick-me`"
      style="padding-left: 10px"
      nav
      dense
      class="mt-0 pt-0"
      expand
    >
      <v-list-item link :to="`/streams`" exact class="">
        <v-list-item-icon>
          <v-icon small class>mdi-arrow-left-drop-circle</v-icon>
        </v-list-item-icon>
        <v-list-item-content>
          <v-list-item-title class="font-weight-bold">Projects</v-list-item-title>
        </v-list-item-content>
      </v-list-item>

      <v-subheader>PROJECT MENU</v-subheader>
      <v-list-item link exact :to="`/streams/${stream.id}`">
        <v-list-item-icon>
          <v-icon small>mdi-home</v-icon>
        </v-list-item-icon>
        <v-list-item-content>
          <v-list-item-title>Project Home</v-list-item-title>
        </v-list-item-content>
      </v-list-item>
      <v-list-item
        v-if="stream.role !== 'stream:reviewer'"
        v-tooltip.bottom="'Create a new branch to help categorise your commits.'"
        link
        @click="newBranchDialog = true"
      >
        <v-list-item-icon>
          <v-icon small style="padding-top: 10px" class="primary--text">
            mdi-plus-box
          </v-icon>
        </v-list-item-icon>
        <v-list-item-content>
          <v-list-item-title>New Branch</v-list-item-title>
          <v-list-item-subtitle class="caption">
            Create a new branch to help categorise your commits.
          </v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>

      <!-- Branch menu group -->
      <v-list-group v-model="branchMenuOpen" class="my-2">
        <template #activator>
          <v-list-item-icon>
            <v-icon small>mdi-source-branch</v-icon>
          </v-list-item-icon>
          <v-list-item-title>Branches ({{ totalBranchCount - 1 }})</v-list-item-title>
        </template>
        <!-- <v-divider class="mb-1"></v-divider> -->

        <div v-if="!loading">
          <template v-for="(item, i) in groupedBranches">
            <div
              :key="Date.now() + i"
              style="
                display: flex;
                flex-direction: row;
                align-content: center;
                justify-content: flex-start;
                align-items: center;
                gap: 20px;
                padding: 0px;
                margin: 0px;
                margin-top: -40px;
                max-height: 50px;
                transform: translate(0px, 53px);
              "
            >
              <div style="height: 40px; transform: translate(10px, 0px)">
                <v-icon medium class="primary--text">mdi-chevron-down</v-icon>
              </div>
              <div>
                <h5 class="text--text">{{ item.displayName }}</h5>
                <p class="text--text caption">{{ item.branches }} Branches</p>
              </div>
            </div>

            <!-- HERE IS THE CONTROL  -->
            <v-list-group
              :key="i"
              sub-group
              :value="item.expand"
              prepend-icon=""
              :group="item.name"
              style="
                transform: translate(-40px);
                /* border-bottom: 1px #d2dad3 solid; */
                padding: 0;
                width: 340px;
              "
            >
              <template>
                <v-list-item style="overflow: visible">
                  <!-- <v-list-item-icon style="position: relative; left: -8px">
                    <v-icon style="padding-top: 10px">
                      {{ item.expand ? 'mdi-chevron-down' : 'mdi-chevron-down' }}
                    </v-icon>
                  </v-list-item-icon>
                  <v-list-item-content style="position: relative; left: -20px">
                    <v-list-item-title>
                      {{ item.displayName }}
                    </v-list-item-title>
                    <v-list-item-subtitle class="caption">
                      {{ item.branches }} Branches
                      {{ item.children }}
                    </v-list-item-subtitle> -->
                  <v-list-item-icon style="position: relative; left: -8px">
                    <v-icon style="padding-top: 10px"></v-icon>
                  </v-list-item-icon>
                  <v-list-item-content
                    style="position: relative; left: -20px; width: 350px"
                  >
                    <v-list-item-title></v-list-item-title>
                    <v-list-item-subtitle class="caption"></v-list-item-subtitle>
                    <!-- HERE GOES THE NESTED ITEMS -->
                    <template v-for="(subitem, x) in item.children">
                      <v-list-group
                        :key="x"
                        :value="subitem.expand"
                        prepend-icon=""
                        :group="subitem.ori"
                        style="
                          transform: translate(-0px, -15px);
                          border: 1px solid #d2dad3;
                          border-radius: 10px;
                          margin-top: 10px;
                          max-width: 100%;
                        "
                      >
                        <template #activator>
                          <v-list-item
                            style="
                              overflow: visible;
                              /* background-color: #e3e3e3; */
                              min-width: 150%;
                              transform: translate(-25px, -0px);
                            "
                          >
                            <v-list-item-content
                              style="position: relative; left: -16px"
                            >
                              <v-list-item-title>
                                {{ subitem.displayOri }}
                              </v-list-item-title>
                              <v-list-item-subtitle class="caption">
                                {{ subitem.children.length }} Branches
                              </v-list-item-subtitle>
                            </v-list-item-content>
                          </v-list-item>
                        </template>
                        <v-list-item
                          v-for="(kid, j) in subitem.children"
                          :key="j"
                          :to="`/streams/${stream.id}/branches/${kid.name}`"
                          exact
                          style="padding-left: 5px"
                        >
                          <v-list-item-icon>
                            <v-icon small style="padding-top: 10px">
                              mdi-source-branch
                            </v-icon>
                          </v-list-item-icon>
                          <v-list-item-content style="position: relative; left: -12px">
                            <v-list-item-title>
                              {{ kid.displayName }} ({{ kid.commits.totalCount }})
                            </v-list-item-title>
                            <v-list-item-subtitle class="caption">
                              <v-icon small>mdi-source-commit</v-icon>
                              {{ kid.displayCommit }}
                            </v-list-item-subtitle>
                          </v-list-item-content>
                        </v-list-item>
                      </v-list-group>
                    </template>
                    <!-- HERE ENDS -->
                  </v-list-item-content>
                </v-list-item>
              </template>
            </v-list-group>
            <v-divider class="mb-2" />
          </template>
        </div>

        <v-skeleton-loader v-else type="list-item-two-line" />
        <!-- <v-divider class="mb-2" /> -->
      </v-list-group>

      <!-- Comments  -->
      <v-list-item link exact :to="`/streams/${stream.id}/comments`">
        <v-list-item-icon>
          <v-icon small>mdi-comment-outline</v-icon>
        </v-list-item-icon>
        <v-list-item-content>
          <v-list-item-title>Comments</v-list-item-title>
        </v-list-item-content>
      </v-list-item>

      <!-- Other menu items go here -->

      <!-- <v-list-item link :to="`/streams/${stream.id}/globals`">
        <v-list-item-icon>
          <v-icon small>mdi-earth</v-icon>
        </v-list-item-icon>
        <v-list-item-content>
          <v-list-item-title>Globals</v-list-item-title>
        </v-list-item-content>
      </v-list-item>
      <portal-target name="stream-globals-nav" /> -->

      <!-- <v-list-item link :to="`/streams/${stream.id}/uploads`">
        <v-list-item-icon>
          <v-icon small>mdi-arrow-up</v-icon>
        </v-list-item-icon>
        <v-list-item-content>
          <v-list-item-title>Import File</v-list-item-title>
        </v-list-item-content>
      </v-list-item> -->

      <!-- <v-list-item link :to="`/streams/${stream.id}/webhooks`">
        <v-list-item-icon>
          <v-icon small>mdi-webhook</v-icon>
        </v-list-item-icon>
        <v-list-item-content>
          <v-list-item-title>Webhooks</v-list-item-title>
        </v-list-item-content>
      </v-list-item> -->

      <v-list-item link :to="`/streams/${stream.id}/collaborators`">
        <v-list-item-icon>
          <v-icon small>mdi-account-group</v-icon>
        </v-list-item-icon>
        <v-list-item-content>
          <v-list-item-title>Collaborators</v-list-item-title>
        </v-list-item-content>
      </v-list-item>

      <v-list-item link :to="`/streams/${stream.id}/settings`">
        <v-list-item-icon>
          <v-icon small>mdi-cog</v-icon>
        </v-list-item-icon>
        <v-list-item-content>
          <v-list-item-title>Settings</v-list-item-title>
        </v-list-item-content>
      </v-list-item>
    </v-list>
    <v-dialog
      v-model="newBranchDialog"
      max-width="500"
      :fullscreen="$vuetify.breakpoint.xsOnly"
    >
      <new-branch
        @close="newBranchDialog = false"
        @refetch-branches="refetchBranches()"
      />
    </v-dialog>
  </portal>
</template>
<script>
import { computed } from 'vue'
import {
  STANDARD_PORTAL_KEYS,
  buildPortalStateMixin
} from '@/main/utils/portalStateManager'
import { StreamEvents } from '@/main/lib/core/helpers/eventHubHelper'
import { useRoute } from '@/main/lib/core/composables/router'
import { useAllStreamBranches } from '@/main/lib/stream/composables/branches'
import { reservedBranchNamesArray } from '../utils/streamDataManager'

export default {
  components: {
    NewBranch: () => import('@/main/dialogs/NewBranch')
  },
  mixins: [buildPortalStateMixin([STANDARD_PORTAL_KEYS.Nav], 'stream-nav', 0)],
  props: {
    stream: {
      type: Object,
      default: () => null
    }
  },
  setup() {
    const route = useRoute()
    const streamId = computed(() => route.params.streamId)
    const { localBranches, refetchBranches, totalBranchCount, branchesLoading } =
      useAllStreamBranches(streamId)

    return {
      localBranches,
      refetchBranches,
      totalBranchCount,
      loading: branchesLoading
    }
  },
  data() {
    return {
      branchMenuOpen: false,
      newBranchDialog: false,
      archTabOpen: false
    }
  },
  computed: {
    groupedBranches() {
      const branches = this.localBranches

      const items = []

      const validBranches = branches.filter(
        (item) => !reservedBranchNamesArray.includes(item.name)
      )
      const allSpecialties = validBranches
        .filter((item) => item.name !== 'main')
        .map((item) => item.name.split('/')[0])
      const uniqueSpecialties = [...new Set(allSpecialties)]

      uniqueSpecialties.forEach((specialty) => {
        const specialtyBranches = branches.filter(
          (branch) => branch.name.split('/')[0] === specialty
        )
        let displaySpecialty = ''
        try {
          displaySpecialty = [
            specialty.charAt(0).toUpperCase(),
            specialty.slice(1)
          ].join('')
        } catch {
          displaySpecialty = ''
        }
        if (displaySpecialty.length <= 3) {
          displaySpecialty = displaySpecialty.toUpperCase()
        }
        const item = {
          name: specialty,
          displayName: displaySpecialty,
          type: 'group',
          children: [],
          expandItem: false,
          branches: 0
        }
        const allOrigins = specialtyBranches
          .filter((item) => item.name !== 'main')
          .map((item) => item.name.split('/')[1])
        const uniqueOrigins = [...new Set(allOrigins)]

        const originArray = uniqueOrigins.map((origin) => ({
          ori: origin,
          expand: false,
          displayOri: [origin.charAt(0).toUpperCase(), origin.slice(1)].join(''),
          children: []
        }))

        item.children = originArray
        items.push(item)
      })

      validBranches.forEach((branch) => {
        const parts = branch.name.split('/')

        let lastCommit = 'No Commits Yet'
        if (branch.commits.items[0] !== undefined) {
          lastCommit = `${branch.commits.items[0].message}`
        }

        const currSpecialty = items.find((item) => item.name === parts[0])
        const currOrigin = currSpecialty.children.find((item) => item.ori === parts[1])

        const branchObj = {
          ...branch,
          // displayName: parts.slice(1).join('/'),
          displayName: parts[2],
          type: 'item',
          //TODO MAYBE INCORPORATE HEALTH
          displayOri: [parts[1].charAt(0).toUpperCase(), parts[1].slice(1)].join(''),
          ori: parts[1],
          displayCommit: lastCommit
        }

        currOrigin.children.push(branchObj)
        if (this.$route.path.includes(branch.name)) currOrigin.expand = true
        if (this.$route.path.includes(branch.name)) currSpecialty.expandItem = true
      })

      items.forEach((item) => {
        let branchCount = 0
        const allOrigins = item.children
        allOrigins.forEach((origin) => {
          branchCount += origin.children.length
        })
        item.branches = branchCount
      })

      return items
    },
    sortedBranches() {
      return [
        this.localBranches.items.find((b) => b.name === 'main'),
        ...this.localBranches.items.filter(
          (b) => b.name !== 'main' && b.name !== 'globals'
        )
      ]
    }
  },
  watch: {
    $route() {
      if (!this.branchMenuOpen)
        this.branchMenuOpen = this.$route.name.toLowerCase().includes('branch')
    }
  },
  mounted() {
    this.branchMenuOpen = this.$route.name.toLowerCase().includes('branch')
    this.$eventHub.$on(StreamEvents.RefetchBranches, async () => {
      await this.refetchBranches()
    })
    this.$eventHub.$on('show-new-branch-dialog', () => {
      this.newBranchDialog = true
    })
  }
}
</script>
