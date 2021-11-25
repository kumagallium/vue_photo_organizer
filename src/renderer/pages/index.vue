<template>
  <v-main wrap>
    <v-container fluid fill-height>
      <v-row>
        <v-spacer></v-spacer>
        <v-col cols="11" sm="11" md="11">
          <v-card flat class="ma-4 pa-4">
            <v-card color="teal lighten-5" class="pa-4 overflow-y-auto"
                      max-height="750">
              <v-row>
                  <v-col cols="3" sm="3" md="3" v-for="(pic, idx) in unq_pictures" :key="idx">
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
                      <v-text-field
                        label="Name"
                        placeholder="Name"
                        v-model="unq_picname[idx].split('.')[0]"
                        outlined
                      ></v-text-field>
                      {{"."+unq_picname[idx].split('.')[1]}}
                      <v-combobox
                        multiple
                        small-chips
                        :items="tags"
                        v-model="selectedtags[pic.split('/')[pic.split('/').length -1]]"
                      ></v-combobox>
                    </v-card>
                  </v-col>
              </v-row>
            </v-card>
          </v-card>
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
      unq_pictures:[],
      unq_picname:[],
      tags: [],
      selectedtags: {},
      directory:""//
    }
  },
  methods: {
    async find() {
      try {
        this.pictures = await globby([this.directory+'**/*.png',this.directory+'**/*.jpg',
                                    this.directory+'**/*.JPG',this.directory+'**/*.jpeg']);
        this.tags = fs.readdirSync(this.directory).filter(dir=>dir.indexOf(".")<0)
        for(let pic of this.pictures){
          var picname = pic.split("/")[pic.split("/").length - 1]
          var tagname = pic.split(this.directory)[1].split("/")[0].split(".")
          if(this.unq_pictures.indexOf(picname)<0){
            this.unq_pictures.push(pic)
            this.unq_picname.push(picname)
          }
          if(tagname.length == 1){
              if(Object.keys(this.selectedtags).indexOf(picname) < 0){
                this.selectedtags[picname] = [tagname[0]]
              }
              else{
                console.log(1)
                console.log(this.selectedtags[picname])
                this.selectedtags[picname].push(tagname[0])
                console.log(this.selectedtags[picname])
              }
          }
        }
          //this.selectedtags
      } catch (err) {
        this.pictures = []
        this.tags = []
        this.selectedtags = {}
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
