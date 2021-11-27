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
                    <v-card>
                      <v-row>
                        <v-col class="pt-0">
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
                        </v-col>
                      </v-row>
                      <v-row class="ma-0">
                        <v-col cols="9" sm="9" md="9" class="pr-0">
                          <v-text-field
                            label="Name"
                            placeholder="Name"
                            v-model="unq_picname[idx].split('.')[0]"
                            outlined
                            dense
                            :hide-details="Object.keys(error).indexOf(String(idx))<0"
                            :error-messages="error[idx]"
                            @change="rename($event,unq_picname[idx],idx)"
                          ></v-text-field>
                        </v-col>
                        <v-col cols="3" sm="3" md="3" class="pl-0">
                          <v-card flat style="line-height:32px;">
                            {{"."+unq_picname[idx].split('.')[1]}}
                          </v-card>
                        </v-col>
                      </v-row>
                      <v-row class="ma-0">
                        <v-col class="py-0">
                          <v-combobox
                            multiple
                            small-chips
                            hide-details
                            :items="tags"
                            v-model="selectedtags[pic.split('/')[pic.split('/').length -1]]"
                             class="py-0"
                          ></v-combobox>
                        </v-col>
                      </v-row>
                      <v-row class="ma-0">
                        <v-col>
                          <div class="body-2 grey--text text--darken-1 text-right">
                            <v-icon small>mdi-clock</v-icon>{{pic_dates[pic.split('/')[pic.split('/').length -1]]}}
                          </div>
                        </v-col>
                      </v-row>
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
var moment = require("moment");
const globby = require('globby');
const fs = require('fs');
const Store = require('electron-store')
const store = new Store()

export default {
  components: {
  },
  data () {
    return {
      pictures: [],
      unq_pictures:[],
      unq_picname:[],
      pic_dates:{},
      tags: [],
      selectedtags: {},
      directory:"",
      error:{}
    }
  },
  methods: {
    async find() {
      console.log("find")
      try {
        this.pictures = await globby([this.directory+'**/*.png',this.directory+'**/*.jpg',
                                    this.directory+'**/*.JPG',this.directory+'**/*.jpeg']);
        this.tags = fs.readdirSync(this.directory).filter(dir=>dir.indexOf(".")<0)
        this.selectedtags = {}
        this.unq_pictures = []
        this.unq_picname = []
        this.pic_dates = []
        for(let pic of this.pictures){
          var picname = pic.split("/")[pic.split("/").length - 1]
          var tagname = pic.split(this.directory)[1].split("/")[0].split(".")
          if(this.unq_picname.indexOf(picname)<0){
            this.unq_pictures.push(pic)
            this.unq_picname.push(picname)
            this.pic_dates[picname] = moment(fs.statSync(pic).birthtime).format('MMM D, YYYY')
          }
          if(tagname.length == 1){
              if(Object.keys(this.selectedtags).indexOf(picname) < 0){
                this.selectedtags[picname] = [tagname[0]]
              }
              else{
                this.selectedtags[picname].push(tagname[0])
              }
          }
        }
        var sortidxs = this.argsort(Object.values(this.pic_dates))
        var unq_pictures_tmp = []
        var unq_picname_tmp = []
        for(var sortidx of sortidxs){
          unq_pictures_tmp.push(this.unq_pictures[sortidx])
          unq_picname_tmp.push(this.unq_picname[sortidx])
        }
        this.unq_pictures = unq_pictures_tmp
        this.unq_picname = unq_picname_tmp
      } catch (err) {
        this.pictures = []
        this.tags = []
        this.selectedtags = {}
      }
    },
    argsort(array) {
      const arrayObject = array.map((value, idx) => { return { value, idx }; });
      arrayObject.sort((a, b) => {
          if (a.value < b.value) {
              return -1;
          }
          if (a.value > b.value) {
              return 1;
          }
          return 0;
      });
      const argIndices = arrayObject.map(data => data.idx);
      return argIndices;
    },
    move(from, to){
      console.log("move")
      fs.rename(from, to, (err) => {
          if (err) throw err;
      });
    },
    rename(event, pic,idx){
      console.log("rename")
      var changedname = event+"."+pic.split(".")[pic.split(".").length-1]
      if(this.unq_picname.indexOf(changedname)<0){
        delete this.error[idx]
        for(let picpath of this.pictures.filter(p=>p.indexOf(pic)>-1)){
          var dirtmp = picpath.replace(pic,"")
          var extension = picpath.split(".")[picpath.split(".").length - 1]
          this.move(picpath,dirtmp + event + "." + extension)
        }
      }
      else{
        if(pic == changedname){
          delete this.error[idx]
        }
        else{
          console.log("else")
          this.unq_picname[idx].split('.')[0] = pic
          this.error[idx] = "This image name already exists."
        }
      }
      this.find()
    }
  },
  mounted(){
    this.directory = store.get('dir')
    this.find();
  }
}
</script>

<style>
.v-select__selections {
  overflow: scroll;
  flex-wrap: nowrap;
}
.v-chip {
  overflow: initial;
}
</style>
