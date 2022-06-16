<template>
  <v-app-bar>
    <v-app-bar-title>
      <v-tooltip location="bottom">
        <template v-slot:activator="{ props }">
          <v-btn
              icon="mdi-home"
              href="/"
              v-bind="props"
          />
        </template>
        <span>Home</span>
      </v-tooltip>
    </v-app-bar-title>
    <slot style="float: right" />
    <v-btn
      prepend-icon="mdi-github"
      href="https://github.com/BlueberryMC/"
      rounded
    >
      GitHub
    </v-btn>
    <v-btn
      prepend-icon="mdi-file-document"
      href="https://docs.blueberrymc.net/"
      rounded
    >
      Documentation
    </v-btn>
    <v-btn
        prepend-icon="mdi-cloud-download"
        href="/"
        rounded
    >
      Downloads
    </v-btn>
  </v-app-bar>
</template>
