<template>
  <v-sheet
    class="fill-height d-flex align-center justify-center"
    color="transparent">
    <span style="font-size: 20px;color:white;">
      loading 
      <span v-if="is_loading_assets">assets</span>
      <span v-if="is_loading_textures">textures</span>
      <span v-if="is_loading_3d_models">3D models</span>
      <span v-if="is_loading_components">components</span>
      ...
    </span>
  </v-sheet>
</template>

<script>
export default {
  data() {
    return {
      is_loading_assets: false,
      is_loading_textures: false,
      is_loading_3d_models: false,
      is_loading_components: false,
    }
  },

  async mounted() {
    const wait = (ms) => new Promise(resolve => setTimeout(resolve, ms))

    // Séquence du loader, un à un
    this.is_loading_assets = true
    await wait(466)
    this.is_loading_assets = false

    this.is_loading_textures = true
    await wait(237)
    this.is_loading_textures = false

    this.is_loading_3d_models = true
    await wait(600)
    this.is_loading_3d_models = false

    this.is_loading_components = true
    await wait(120)
    this.is_loading_components = false

    // Notifie le parent que le loader est terminé
    this.$emit('loading-finished')
  }
}
</script>
