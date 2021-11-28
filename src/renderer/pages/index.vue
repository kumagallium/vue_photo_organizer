<template>
  <v-main wrap>
    <v-container fluid fill-height>
      <v-row>
        <v-spacer></v-spacer>
        <v-col cols="12" sm="12" md="12">
          <v-card flat class="ma-0 pa-0">
            <v-card flat class="pa-5 overflow-y-auto">
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
                            @change="changetag($event,unq_picname[idx],idx)"
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
import { utimes } from 'utimes';
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
      selectedtags_save: {},
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
        this.tags = fs.readdirSync(this.directory).filter(dir=>(dir.indexOf(".")<0)&&(dir!="trashbox"))
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
        this.selectedtags_save = JSON.parse(JSON.stringify(this.selectedtags))
      } catch (err) {
        this.pictures = []
        this.tags = []
        this.selectedtags = {}
      }
    },
    argsort(array) {
      const arrayObject = array.map((value, idx) => { return { value, idx }; });
      arrayObject.sort((a, b) => {
          if ( new Date(a.value) < new Date(b.value)) {
              return -1;
          }
          if ( new Date(a.value) > new Date(b.value)) {
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
    copy(from, to){
      console.log("copy")
      fs.copyFile(from, to, (err) => {
        if (err) throw err;
      });
    },
    delete(filepath){
      fs.unlink(filepath, (err) => {
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
    },
    mkdir(dirpath){
      fs.mkdir(dirpath, (err) => {
        if (err) { throw err; }
      });
    },
    deletedir(dirpath){
      fs.rmdir(dirpath, (err) => {
          if (err) throw err;
      });
    },
    changetag(event, pic,idx){
      console.log("changetag")
      if(Object.keys(this.selectedtags_save).indexOf(pic)>-1){
        var delete_tag = this.selectedtags_save[pic].filter(sts=>this.selectedtags[pic].indexOf(sts)<0)
        var add_tag = this.selectedtags[pic].filter(st=>this.selectedtags_save[pic].indexOf(st)<0)
      }
      else{
        var delete_tag = []
        var add_tag = event
      }
      if(add_tag.length > 0){
        if(fs.readdirSync(this.directory).indexOf(add_tag)<0){
          this.mkdir(this.directory+add_tag)
        }
        var makedir = this.directory+add_tag
        var dirpath = this.unq_pictures.filter(upic=>upic.indexOf(pic)>0)[0]
        if(fs.readdirSync(this.directory).filter(p=>p.indexOf(pic)>-1).length > 0){
          var stat = fs.statSync(dirpath);
          this.move(dirpath,makedir+"/"+pic)
          utimes(makedir+"/"+pic, {btime: stat.birthtime.getTime()});
        }
        else{
          var stat = fs.statSync(dirpath);
          this.copy(dirpath,makedir+"/"+pic)
          utimes(makedir+"/"+pic, {btime: stat.birthtime.getTime()});
        }
      }
      if(delete_tag.length > 0){
        console.log("delete",this.pictures.filter(p=>p.indexOf(pic)>-1))
        if(this.pictures.filter(p=>p.indexOf(pic)>-1).length > 1){
          var dirpath = this.unq_pictures.filter(upic=>upic.indexOf(pic)>0)[0]
          this.delete(this.directory+delete_tag+"/"+pic)
        }
        else{
          var dirpath = this.unq_pictures.filter(upic=>upic.indexOf(pic)>0)[0]
          var stat = fs.statSync(dirpath);
          this.move(dirpath,this.directory+pic)
          utimes(this.directory+pic, {btime: stat.birthtime.getTime()});
        }
      }
      for(var tag of this.tags){
        if(fs.readdirSync(this.directory+tag).length == 0){
          this.deletedir(this.directory+tag)
        }
      }
      this.find()
    }
  },
  mounted(){
    this.directory = store.get('dir')
    if(fs.readdirSync(this.directory).indexOf("trashbox")<0){
      console.log("makedir")
      console.log(this.directory+"trashbox")
      this.mkdir(this.directory+"trashbox")
    }
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
