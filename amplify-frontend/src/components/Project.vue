<template>
  <div class="project" v-if="name">
    <h2>{{name}}</h2>
    <p>
      Delete project
      <Icon 
        class="deleteButton"
        name="trash-alt"
        v-on:click="deleteProject(name)"
        scale="1" 
      /> 
    </p>
    
    <ImageUpload
      @imageData="newImage"
      v-bind:clear="clearImage"
    />

    <PageSelector 
      v-bind:page="pageNumber"
      @update="selectPage"
    />

    <a v-bind:style="{ color: uploadActiveColor }" v-on:click="upload(pageNumber)" >
      Upload
    </a>

    <p>
      <a v-bind:style="{ color: uploadActiveColor }" v-on:click="generatePDF(name)" >
        Generate PDF
      </a>
    </p>

  </div>
</template>

<script>
import Icon from 'vue-awesome/components/Icon'
import PageSelector from './PageSelector.vue'
import ImageUpload from './ImageUpload.vue'
import { Storage } from 'aws-amplify'
import 'vue-awesome/icons/trash-alt'

export default {
  name: 'Project',
  components: {
    ImageUpload,
    PageSelector,
    Icon
  },
  data() {
    return {
      proj: null,
      name: null,
      pageNumber: 1,
      imageData: null,
      clearImage: null,
      uploadActiveColor: '#f90'
    }
  },
  props: ['project', 'count', 'username'],
  watch: { 
    project: function(val) {
      if(!val){
        this.name = null
        this.proj = null
        this.pageNumber = 1
      }else {
        this.proj = val;
        this.name = val.project_name;
      }
    },
    count: function(val) {
      this.pageNumber = val + 1
    }
  },
  methods: {
    upload: function (page) {
      if (!this.imageData) { return }
      let fileName = `${this.username}/${this.name}/${page}.png`
      this.uploadActiveColor = '#c5c7c5';

      Storage.put(fileName, this.imageData, {
          level: 'private',
          contentType: 'image/png'
      })
      .then (result => {
        console.log(result)
        this.clearImage = result;
        this.uploadActiveColor = '#f90';
        this.pageNumber = this.pageNumber + 1
      })
      .catch(err => console.log(err));
    },
    newImage: function (blob) {
      this.imageData = blob
    },
    selectPage: function (val) {
      this.clearImage = val
      this.pageNumber = val
      this.imageData = null
    },
    deleteProject: function (name) {
      if(confirm('Are you sure?')){
        this.$emit('delete', name)
      }
    },
    generatePDF: function (name) {
      this.$emit('generate', name)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->

<style scoped>

.project {
  margin: 15px;
}

.pageSelector {
  margin: 15px;
}

.uploadButton {
  color: #f90;
}


</style>