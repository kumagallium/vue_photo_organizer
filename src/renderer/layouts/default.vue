<template>
  <v-app>
    <Navi></Navi>
    <nuxt />
  </v-app>
</template>

<script>
import Navi from '@/components/Navi'
export default {
  components: {Navi }
}
</script>

<style>
body {
  margin: 0 !important;
}
</style>
