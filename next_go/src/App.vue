<template>
  <v-app>
    <v-main>
      <!-- Loader -->
      <BeforeLoading v-if="is_before_loading" @loading-finished="handleLoadingFinished" />

      <!-- Composant suivant -->
      <WhatIsYourName v-if="is_before_loading === false" />
    </v-main>
  </v-app>
</template>

<script>
import BeforeLoading from './utils/vue/beforeLoading.vue'
import WhatIsYourName from './components/whatIsYourName.vue'

// Initialisation du localStorage si jamais il n'existe pas
if (!localStorage.getItem("currently_loading")) {
  localStorage.setItem("currently_loading", "true")
}

export default {
  components: {
    BeforeLoading,
    WhatIsYourName
  },

  data() {
    return {
      is_before_loading: localStorage.getItem("currently_loading") === "true",
    }
  },

  methods: {
    handleLoadingFinished() {
      this.is_before_loading = false
      localStorage.setItem("currently_loading", "false")
    }
  }
}
</script>

