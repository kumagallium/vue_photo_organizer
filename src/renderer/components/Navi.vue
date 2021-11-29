<template>
  <v-navigation-drawer
    clipped
    permanent
    app
    color="teal lighten-1"
    dark
    width="120"
  >

    <v-list-item style="background-color:#004D40;">
        <v-list-item-content style="text-align:center;">
            <nuxt-link to="/">
                <v-img
                max-height="35"
                max-width="35"
                src="VPO.png"
                class="mx-auto my-2"
                ></v-img>
            </nuxt-link>
        </v-list-item-content>
    </v-list-item>
    <v-divider></v-divider>

    <v-list dense flat>
      <v-list-item-group color="amber">
        <v-list-item :to="'/'">
          <v-list-item-title># All</v-list-item-title>
        </v-list-item>
        <v-list-item
          :exact="true"
        >
            <v-list-item-content>
                <v-list-item-title># Unorganized</v-list-item-title>
            </v-list-item-content>
        </v-list-item>
        <v-list-item
          v-for="tag in tags"
          :key="tag"
          :exact="true"
          link
        >
            <v-list-item-content>
                <v-list-item-title># {{ tag }}</v-list-item-title>
            </v-list-item-content>
        </v-list-item>
      </v-list-item-group>
    </v-list>

    <v-list dense flat style="position:absolute;bottom:0;"><v-list-item
          :exact="true"
        >
        </v-list-item>
      <v-list-item-group>
        <v-list-item :to="'/settings'" color="amber">
          <v-list-item-icon class="mr-3">
            <v-icon medium >mdi-cog</v-icon>
          </v-list-item-icon>
          <v-list-item-title>Settings</v-list-item-title>
        </v-list-item>
      </v-list-item-group>
    </v-list>
  </v-navigation-drawer>
</template>

<script>

export default {
  computed: {
  },
  props: ['tags'],
  methods: {
  },
}
</script>
