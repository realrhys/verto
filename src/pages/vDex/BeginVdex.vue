<template>
  <q-page class="flex flex-center text-white bg-black">
    <q-card class="bg-black" style="max-width: 500px;" flat>
      <q-card-section>
        <div class="text-h6">
          This is the Begin VTX stuff. If not url is available, we will need to redirect, else we can redirect into another place.
        </div>
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script>
export default {
  data () {
    return {}
  },
  mounted () {
    const data = this.$store.state.currentwallet.config.data
    if (data && data.vdex && data.vdex.url) {
      this.$router.push({ path: 'vdex-options' })
    } else {
      this.$router.push({ path: 'vdex-install-instructions' })
    }
  },
  methods: {
  }
}
</script>

<style>
</style>
