<template>
    <v-container fluid class="pa-0">
        <v-row justify="center">
          <v-col>
            <v-text-field
              label="Directory"
              placeholder="Directory"
              @change="find"
              v-model="directory"
              outlined
          ></v-text-field>
          </v-col>
        </v-row>
        <v-row justify="center">
            <v-col class="pt-8" cols="3" sm="3" md="3" v-for="pic in pictures" :key="pic">
              <v-card class="px-md-6 px-sm-6 px-3">
                <v-img :src="'file://'+pic"/>
                {{pic}}
              </v-card>
            </v-col>
        </v-row>
    </v-container>
      
</template>

<script>
//import { remote } from 'electron'
const globby = require('globby');

export default {
  components: {
  },
  data () {
    return {
      pictures: [],
      directory:''
    }
  },
  methods: {
    async find() {
      this.pictures = await globby([this.directory+'*.png',this.directory+'*.jpg']);
    }
  },
  mounted(){
    this.find() 
    console.log(this.pictures);
  }
}
</script>

<style>
</style>
