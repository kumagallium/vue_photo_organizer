<template>
  <v-main wrap>
    <v-container fluid fill-height>
      <v-row>
        <v-spacer></v-spacer>
        <v-col cols="11" sm="11" md="11">
            Settings
            <v-text-field
                label="Directory"
                placeholder="Directory"
                @change="setdir"
                v-model="directory"
                outlined
            ></v-text-field>
        </v-col>
      </v-row>
    </v-container>
  </v-main>
</template>


<script>
const globby = require('globby');
const fs = require('fs');
const Store = require('electron-store')
const store = new Store()

export default {
    components: {
    },
    data () {
        return {
        files: [],
        tags: [],
        directory:''
        }
    },
    methods: {
        async setdir() {
            //this.files = fs.readdirSync(this.directory).filter(dir=>dir.indexOf(".")<0)//await globby([this.directory+'*.png',this.directory+'*.jpg',this.directory+'*.JPG',this.directory+'*.jpeg']);
            //store.set('dir', '🦄');
            //console.log(store.get('dir'));
            //=> '🦄'

            // Use dot-notation to access nested properties
            store.set('dir', this.directory);
            console.log(store.get('dir'));
            this.tags = fs.readdirSync(this.directory).filter(dir=>(dir.indexOf(".")<0)&&(dir!="trashbox"))
            //=> {bar: true}

            //store.delete('unicorn');
            //console.log(store.get('unicorn'));
            //=> undefined
            this.$nuxt.$emit('gettags', this.tags)
        }
    },
    mounted(){
      console.log((store.get('dir')!=undefined)&&(store.get('dir')!=""))
      try{
        this.directory = store.get('dir')
        this.tags = fs.readdirSync(this.directory).filter(dir=>(dir.indexOf(".")<0)&&(dir!="trashbox"))
        this.$nuxt.$emit('gettags', this.tags)
      }
      catch (error) {
          this.directory = "/Users/"
      }
    }
}
</script>