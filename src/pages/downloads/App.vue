<script setup lang="ts">
import NavigationBar from '@/components/NavigationBar.vue'
import { oneLine } from '@/util'
</script>

<template>
  <v-app>
    <NavigationBar/>
    <v-main>
      <div class="main-wrap2">
        <!-- Versions -->
        <div v-if="downloaded" class="version-list">
          <v-list v-if="downloaded" max-width="300px" height="100%" max-height="100%">
            <!--suppress JSValidateTypes, JSIncompatibleTypesComparison -->
            <template
              v-for="(download, index) in downloads"
              :key="download.version_group"
            >
              <!-- Version entry -->
              <v-list-item
                v-if="versionGroups[download.project].find(({ name }) => name === download.version_group)?.legacy !== true || showLegacy"
                :title="download.projectCapitalized + ' ' + download.title"
                :subtitle="download.version_group + '.x (' + downloadItems.versions[download.project + '|' + download.version_group].builds.length + ' build' + (downloadItems.versions[download.project + '|' + download.version_group].builds.length === 1 ? '' : 's') + ')'"
                :active="selectedVersion === index"
                :theme="getThemeForVersionGroup(versionGroups[download.project].find(({ name }) => name === download.version_group))"
                @click="selectedVersion = index"
              />
            </template>
          </v-list>
        </div>
        <!-- Contents -->
        <div class="container-wrapper">
          <v-container>
            <!-- Show "Loading..." if data is not ready yet -->
            <div v-if="!downloaded">
              <p>Loading...</p>
              <v-progress-linear
                  :indeterminate="true"
                  color="cyan"
              ></v-progress-linear>
            </div>
            <!-- Show contents if data is ready -->
            <div v-else>
              <!-- Whether to show legacy (unsupported) versions -->
              <v-checkbox
                v-model="showLegacy"
                label="Show unsupported versions"
              />
              <!-- Title -->
              <h1>Downloads for {{ downloads[selectedVersion].projectCapitalized }} {{ downloads[selectedVersion].version_group }}.x</h1>
              <!-- Description -->
              <p>{{ getSelectedVersionGroup()?.description }}</p>
              <!-- Alert if experimental -->
              <v-alert
                v-if="getSelectedVersionGroup()?.experimental === true"
                color="yellow-darken-2"
                type="warning"
                title="Experimental builds"
                style="margin: 10px 0 10px 0;"
              >
                This version is not yet stable, and should NOT be used in production.
              </v-alert>
              <!-- Alert if legacy (unsupported) -->
              <v-alert
                v-if="getSelectedVersionGroup()?.legacy === true"
                color="red-darken-3"
                icon="mdi-information"
                type="error"
                title="Unsupported builds"
                style="margin: 10px 0 10px 0;"
              >
                This version is <b>NOT</b> supported and you will <b>not</b> receive support for these builds.
              </v-alert>
              <!-- General information -->
              <v-alert
                type="info"
                title="Heads up!"
                style="margin: 10px 0 10px 0;"
              >
                Download button with yellow color means that the build is not stable (experimental) and <b>should not be used in production</b>.
                However, if the button is green, it means that the build is <b>promoted</b> and can be used safely in production.
              </v-alert>
              <div class="build-cards">
                <!-- TODO: @SuppressWarnings("DuplicateCode") -->
                <!-- Promoted build card -->
                <template v-if="downloadItems.versions[downloads[selectedVersion].project + '|' + downloads[selectedVersion].version_group].promoted !== null">
                  <v-card
                    v-for="build in [downloadItems.versions[downloads[selectedVersion].project + '|' + downloads[selectedVersion].version_group].promoted]"
                    :key="build.id"
                    width="400px"
                    prepend-icon="mdi-star"
                    :title="'Promoted build (#' + build.build_number + ')'"
                    :subtitle="'Version: ' + build.version"
                  >
                    <template v-slot:text>
                      <div v-if="build.changes.length > 0">
                        <p
                          v-for="change in build.changes"
                          :key="change.id"
                        >
                          <a :href="projects.find(({ name }) => name === downloads[selectedVersion].project).repo_url + '/commit/' + change.sha">
                            {{ change.sha.substring(0, 7) }}
                          </a>
                          <span>: </span>
                          <span>{{ oneLine(change.description) }}</span>
                        </p>
                      </div>
                      <p v-else>
                        <span>No changes</span>
                      </p>
                    </template>

                    <v-card-actions style="align-content: center">
                      <!-- Download button -->
                      <v-btn
                        v-if="build.files.findIndex(file => file.type === 'universal-installer') !== -1"
                        :href="build.files.find(file => file.type === 'universal-installer').download_url"
                        prepend-icon="mdi-cloud-download"
                        class="download-button bg-green-lighten-2"
                      >
                        Download
                      </v-btn>
                      <!-- Commit -->
                      <v-btn
                        v-if="build.changes.length > 0"
                        :href="projects.find(({ name }) => name === downloads[selectedVersion].project).repo_url + '/commit/' + build.changes[0].sha"
                        prepend-icon="mdi-github"
                        class="download-button bg-green-lighten-2"
                      >
                        GitHub
                      </v-btn>
                    </v-card-actions>
                  </v-card>
                </template>
                <!-- Latest build card -->
                <template v-if="downloadItems.versions[downloads[selectedVersion].project + '|' + downloads[selectedVersion].version_group].latest !== null">
                  <v-card
                    v-for="build in [downloadItems.versions[downloads[selectedVersion].project + '|' + downloads[selectedVersion].version_group].latest]"
                    :key="build.id"
                    width="400px"
                    prepend-icon="mdi-code-tags"
                    :title="'Latest build (#' + build.build_number + ')'"
                    :subtitle="'Version: ' + build.version"
                  >
                    <template v-slot:text>
                      <div v-if="build.changes.length > 0">
                        <p
                          v-for="change in build.changes"
                          :key="change.id"
                        >
                          <a :href="projects.find(({ name }) => name === downloads[selectedVersion].project).repo_url + '/commit/' + change.sha">
                            {{ change.sha.substring(0, 7) }}
                          </a>
                          <span>: </span>
                          <span>{{ oneLine(change.description) }}</span>
                        </p>
                      </div>
                      <p v-else>
                        <span>No changes</span>
                      </p>
                    </template>

                    <v-card-actions style="align-content: center">
                      <!-- Download button -->
                      <v-btn
                        v-if="build.files.findIndex(file => file.type === 'universal-installer') !== -1"
                        :href="build.files.find(file => file.type === 'universal-installer').download_url"
                        prepend-icon="mdi-cloud-download"
                        class="download-button"
                      >
                        Download
                      </v-btn>
                      <!-- Commit -->
                      <v-btn
                        v-if="build.changes.length > 0"
                        :href="projects.find(({ name }) => name === downloads[selectedVersion].project).repo_url + '/commit/' + build.changes[0].sha"
                        prepend-icon="mdi-github"
                        class="download-button"
                      >
                        GitHub
                      </v-btn>
                    </v-card-actions>
                  </v-card>
                </template>
              </div>
              <!-- List of downloadable builds -->
              <v-expansion-panels>
                <v-expansion-panel title="Individual builds">
                  <v-expansion-panel-text>
                    <v-list id="itemList">
                      <v-list-item
                        v-for="build in downloadItems.versions[downloads[selectedVersion].project + '|' + downloads[selectedVersion].version_group].builds"
                        :key="build.id"
                        min-height="50px"
                      >
                        <!-- TODO: this (:color) sucks -->
                        <!-- Download button -->
                        <v-btn
                          v-if="build.files.findIndex(file => file.type === 'universal-installer') !== -1"
                          :href="build.files.find(file => file.type === 'universal-installer').download_url"
                          prepend-icon="mdi-cloud-download"
                          class="download-button"
                          :color="(getSelectedVersionGroup()?.experimental === true || build.experimental) ? 'yellow-darken-2' : (getSelectedVersionGroup()?.legacy === true ? 'red-darken-3' : (build.promoted ? 'green-lighten-2' : undefined))"
                          rounded
                        >
                          #{{ build.build_number }}
                        </v-btn>
                        <!-- Changelogs -->
                        <div v-if="build.changes.length > 0">
                          <p
                            v-for="change in build.changes"
                            :key="change.id"
                          >
                            <a :href="projects.find(({ name }) => name === downloads[selectedVersion].project).repo_url + '/commit/' + change.sha">
                              {{ change.sha.substring(0, 7) }}
                            </a>
                            <span>: </span>
                            <span>{{ oneLine(change.description) }}</span>
                          </p>
                        </div>
                        <p v-else>
                          <span>No changes</span>
                        </p>
                        <div style="margin-left: auto">Version: {{ build.version }}</div>
                      </v-list-item>
                    </v-list>
                  </v-expansion-panel-text>
                </v-expansion-panel>
              </v-expansion-panels>
            </div>
          </v-container>
        </div>
      </div>
    </v-main>
  </v-app>
</template>

<script lang="ts">
// Type defs
type Build = {
  id: number
  version_id: number
  build_number: number
  experimental: boolean
  promoted: boolean
  changes: {
    id: number
    build_id: number
    sha: string
    description: string
  }[]
  files: {
    id: number,
    build_id: number
    type: string
    download_url: string
  }[]
  version: string | null
}

type DownloadItem = {
  latest: Build | null
  promoted: Build | null
  builds: Build[]
}

// Imports
import { defineComponent, ref } from 'vue'

// Constants
const downloads = [
  {
    project: 'blueberry',
    projectCapitalized: 'Blueberry',
    version_group: '1.19',
    title: '1.19',
  },
  {
    project: 'blueberry',
    projectCapitalized: 'Blueberry',
    version_group: '1.18',
    title: '1.18.2',
  },
]

// Data
const downloaded = ref(false)

const selectedVersion = ref(0)

const downloadItems: { downloaded: boolean, versions: { [project_version_group: string]: DownloadItem } } = {
  downloaded: false,
  versions: {},
}

const projects: Array<{ id: number, name: string, description: string, created_at: number, repo_url: string }> = []

const versionGroups: {
  [project: string]: Array<
    {
      id: number
      project_id: number
      name: string
      description: string
      created_at: number
      experimental: boolean
      legacy: boolean
      branch: string
    }
  >
} = {}

export default defineComponent({
  name: 'App',

  data: () => ({
    selectedVersion,
    downloads,
    downloadItems,
    downloaded,
    projects,
    versionGroups,
    showLegacy: false,
  }),

  created() {
    const promises = []
    const projectsPromise = fetch('https://api.blueberrymc.net/v1/projects')
      .then((res) => {
        if (res.ok) {
          return res.json()
        }
        throw new Error('Failed to fetch projects')
      })
      .then((res) => {
        projects.push(...res.projects)
      })
    promises.push(projectsPromise)
    downloads.map(({ project }) => project).filter((v, i, a) => a.indexOf(v) === i).forEach((project) => {
      const versionGroupsPromise = fetch(`https://api.blueberrymc.net/v1/projects/${project}/version_groups`)
        .then((res) => {
          if (res.ok) {
            return res.json()
          }
          throw new Error('Failed to fetch version groups for ' + project)
        })
        .then((res) => {
          versionGroups[project] = res.version_groups
        })
      promises.push(versionGroupsPromise)
    })
    for (const download of downloads) {
      const promise = fetch(`https://api.blueberrymc.net/v1/projects/${download.project}/version_groups/${download.version_group}/builds`)
        .then((res) => {
          if (res.status === 200) {
            return res.json()
          } else {
            throw new Error('Failed to fetch builds for ' + download.version_group)
          }
        })
        .then((res: { builds: Build[] }) => {
          const data: DownloadItem = {
            latest: null,
            promoted: null,
            builds: [],
          }
          for (const build of res.builds.reverse()) { // latest = first
            if (build.files.length === 0) {
              // no downloadable files, skip it.
              continue
            }
            if (data.latest === null && !build.experimental) {
              data.latest = build
            }
            if (data.promoted === null && build.promoted) {
              data.promoted = build
            }
            data.builds.push(build)
          }
          downloadItems.versions[download.project + '|' + download.version_group] = data
        })
        .catch(() => {
          downloadItems.versions[download.project + '|' + download.version_group] = { latest: null, promoted: null, builds: [] }
        })
      promises.push(promise)
    }
    Promise.all(promises).then(() => {
      downloaded.value = true
    }).catch(() => {
      downloaded.value = true
    })
  },

  methods: {
    getSelectedVersionGroup() {
      const download = downloads[selectedVersion.value]
      return versionGroups[download.project].find(({ name }) => name === download.version_group)
    },
    getThemeForVersionGroup: (group: { legacy: boolean, experimental: boolean }) => {
      if (!group) {
        return undefined
      }
      if (group.legacy) {
        return 'red'
      }
      if (group.experimental) {
        return 'yellow'
      }
      return undefined
    },
  },
})
</script>

<style scoped>
/* tried to make independent scroll bar */
.version-list {
  /*
  overflow-x: hidden;
  overflow-y: scroll;
  */
  height: 100%;
}

.container-wrapper {
  /*
  overflow-x: hidden;
  overflow-y: scroll;
  */
  height: 100%;
  flex: 1;
}

.main-wrap2 {
  display: flex;
  height: 100%;
  max-height: 100%;
}

.download-button {
  color: #fff;
  background-color: #00bcd4;
  margin-right: 8px;
}

/*noinspection CssUnusedSymbol*/
.v-theme--red {
  background-color: #410000;
}

/*noinspection CssUnusedSymbol*/
.v-theme--red:hover {
  filter: opacity(80%);
}

/*noinspection CssUnusedSymbol*/
.v-theme--yellow {
  background-color: #636100;
}

/*noinspection CssUnusedSymbol*/
.v-theme--yellow:hover {
  filter: opacity(80%);
}

#itemList>div:nth-child(even) {
  background-color: rgba(255, 255, 255, 0.05);
}

.build-cards {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
  max-width: 100%;
  margin: 15px;
}

.build-cards>div {
  margin: 15px;
}
</style>
