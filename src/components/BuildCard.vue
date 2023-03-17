<script setup lang="ts">
import { oneLine } from '@/util'
</script>

<template>
  <v-card
      v-if="build !== null"
      width="400px"
      :prepend-icon="icon"
      :title="title + ' (#' + build.build_number + ')'"
      :subtitle="'Version: ' + build.version"
  >
    <template v-slot:text>
      <div v-if="build.changes.length > 0">
        <p
            v-for="change in build.changes"
            :key="change.id"
        >
          <a :href="repoUrl + '/commit/' + change.sha">
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
          :theme="theme"
          :color="color"
          class="download-button"
      >
        Download
      </v-btn>
      <!-- Commit -->
      <v-btn
          v-if="build.changes.length > 0"
          :href="repoUrl + '/commit/' + build.changes[0].sha"
          prepend-icon="mdi-github"
          :theme="theme"
          :color="color"
          class="download-button"
      >
        GitHub
      </v-btn>
    </v-card-actions>
  </v-card>
</template>

<script lang="ts">
export default {
  props: {
    title: {
      type: String,
      required: true,
    },
    build: {
      type: Object,
      required: true,
    },
    repoUrl: {
      type: String,
      required: true,
    },
    icon: {
      type: String,
      required: false,
    },
    theme: {
      type: String,
      required: false,
    },
    color: {
      type: String,
      required: false,
    },
  },
}
</script>

<style scoped>
.download-button {
  color: #fff;
  background-color: #00bcd4;
  margin-right: 8px;
}

/*noinspection CssUnusedSymbol*/
.v-theme--yellow {
  background-color: #fbc02d;
}

/*noinspection CssUnusedSymbol*/
.v-theme--yellow:hover {
  filter: opacity(80%);
}

/*noinspection CssUnusedSymbol*/
.v-theme--green {
  background-color: #269a48;
}

/*noinspection CssUnusedSymbol*/
.v-theme--green:hover {
  filter: opacity(80%);
}
</style>
