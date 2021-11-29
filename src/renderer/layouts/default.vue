<template>
  <v-app>
    <Navi :tags="tags"></Navi>
    <nuxt />
  </v-app>
</template>

<script>
import Navi from '@/components/Navi'
export default {
  components: { Navi },
  data () {
    return {
      tags: [],
    }
  },
  created() {
    this.setListener()
  },
  methods: {
    setListener() {
      this.$nuxt.$on('gettags', this.gettags)
    },
    gettags(val) {
      this.tags = val
    }
  }
}
</script>

<style>
body {
  margin: 0 !important;
}
</style>
