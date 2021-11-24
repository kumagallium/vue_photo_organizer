<template>
  <v-navigation-drawer
    clipped
    permanent
    app
    color="rgba(0,77,64,0.9)"
    dark
    width="60"
  >

    <v-list dense flat>
      <v-list-item-group color="amber">
        <v-list-item :to="''">
          <v-list-item-icon>
            <v-icon medium>mdi-home</v-icon>
          </v-list-item-icon>
        </v-list-item>
      </v-list-item-group>
    </v-list>

    <v-list dense flat style="position:absolute;bottom:0;">
      <v-list-item-group>
        <v-list-item to="''" color="white">
          <v-list-item-icon>
            <v-icon medium >mdi-cog</v-icon>
          </v-list-item-icon>
        </v-list-item>
      </v-list-item-group>
    </v-list>
  </v-navigation-drawer>
</template>

<script>

export default {
  computed: {
  },
  methods: {
  },
}
</script>
