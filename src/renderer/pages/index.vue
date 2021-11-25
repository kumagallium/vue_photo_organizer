<template>
  <v-main wrap>
    <v-container fluid fill-height>
      <v-row>
        <v-spacer></v-spacer>
        <v-col cols="11" sm="11" md="11">
          <v-card flat class="ma-4 pa-4">
            <v-row>
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
            <v-card color="teal lighten-5" class="pa-4 overflow-y-auto"
                      max-height="750">
              <v-row>
                  <v-col cols="3" sm="3" md="3" v-for="pic in pictures" :key="pic">
                    <v-card
                    >
                      <v-img
                        aspect-ratio="1"
                        lazy-src="https://picsum.photos/id/11/100/60"
                       :src="'file://'+pic"
                      >
                        <template v-slot:placeholder>
                          <v-row
                            class="fill-height ma-0"
                            align="center"
                            justify="center"
                          >
                            <v-progress-circular
                              indeterminate
                              color="grey lighten-5"
                            ></v-progress-circular>
                          </v-row>
                        </template>
                      </v-img>
                    </v-card>
                  </v-col>
              </v-row>
            </v-card>
          </v-card>
          {{this.files}}
        </v-col>
        <v-spacer></v-spacer>
      </v-row>
    </v-container>
  </v-main>
      
</template>

<script>
//import { remote } from 'electron'
const globby = require('globby');
const fs = require('fs');
const Store = require('electron-store')
const store = new Store()
console.log(store.path);

export default {
  components: {
  },
  data () {
    return {
      pictures: [],
      files: [],
      directory:""//
    }
  },
  methods: {
    async find() {
      try {
        this.pictures = await globby([this.directory+'**/*.png',this.directory+'**/*.jpg',
                                    this.directory+'**/*.JPG',this.directory+'**/*.jpeg']);
        this.files = fs.readdirSync(this.directory).filter(dir=>dir.indexOf(".")<0)
      } catch (err) {
        this.pictures = []
        this.files = []
        //console.log(err)
        // Here you get the error when the file was not found,
        // but you also get any other error
      }
    }
  },
  mounted(){
    this.directory = store.get('dir')
    this.find();
    console.log(this.pictures);
  }
}
</script>

<style>
</style>
