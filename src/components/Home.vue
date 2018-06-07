<template>
  <div class="hello">
    <h5>API says:</h5>
    <h1>{{ greeting }}</h1>
    <hr>
      <div class="p-4 row">
        <div class="col card p-0" v-for="g in galleries" :key="g.id">
          <div class="card-header">{{ g.title }}</div>
          <div class="card-body">
            <img :src="'https://static.dev.local/'+p.path" v-for="p in g.pictures"
              :style="{maxWidth: '64px', maxHeight: '64px', display: 'inline-block'}"/>
          </div>
        </div>
      </div>

        <form-wizard title="Create new gallery"
          subtitle="" step-size="sm" color="#41b883"
          shape="tab"
          @on-complete="onComplete"
          @on-loading="setLoading"
          @on-validate="handleValidation"
          @on-error="handleErrorMessage">
          <tab-content title="Chose a gallery name"
            :before-change="submitData">
            <div class="from-group">
              <label>Name:</label>
              <input type="text" v-model="title" class="form-control">
            </div>
          </tab-content>
          <tab-content title="Upload pictures" :before-change="submitPictures">
            <picture-uploader @change="onFilesSelected" ref="uploader" />
            {{ picturesEndpoint }}
          </tab-content>
          <tab-content title="Done!">
            Yuhuuu! This seems pretty damn simple
          </tab-content>
        </form-wizard>

<!--         <div class="mf__dropper text-center py-5 px-2">
          <svg viewBox="0 0 32 32" height="128"
    width="128" fill="none" stroke="#CCC" stroke-linecap="round" stroke-linejoin="round" stroke-width="2">
        <path d="M9 22 C0 23 1 12 9 13 6 2 23 2 22 10 32 7 32 23 23 22 M11 18 L16 14 21 18 M16 14 L16 29" />
    </svg>
          <p class="lead m-0">
            Drag & drop files here <br> or
          </p>
          <a href="#" class="btn btn-success">Browse for images</a>
        </div> -->



    <!-- <form>
      <label for="gallery-name-input">Gallery name</label>
      <div>
        <input type="text" v-model="title" id="gallery-name-input">
      </div>
    </form>

    <div id="picture-uploader">
      <picture-uploader @change="onFilesSelected" ref="uploader">
      </picture-uploader>
    </div>

    <div>
      <a href="#" class="btn btn-success btn-large">Create gallery</a>
    </div>

    <hr>

    <template v-if="galleries.length === false">
      <div v-for="img in galleries">
        <img :src="img.src" width="250">
      </div>
    </template> -->
  </div>
</template>

<script>
import axios from 'axios'
import PictureUploader from './PictureUploader'
import {FormWizard, TabContent} from 'vue-form-wizard'
import 'vue-form-wizard/dist/vue-form-wizard.min.css'

export default {
  name: 'Home',
  data () {
    return {
      greeting: 'Fetching from /api...',
      title: '',
      picturesEndpoint: {
        location: null,
        method: null
      },
      galleries: []
    }
  },
  methods: {
    onComplete () {
      this.fetchGalleries()
      console.log('onComplete')
    },
    setLoading () {},
    handleValidation: function(isValid, tabIndex){
       console.log('handleValidation Tab: '+tabIndex+ ' valid: '+isValid)
    },
    handleErrorMessage: function(errorMsg){
      console.log('handleErrorMessage', errorMsg)
    },
    submitData () {
      return axios.post('/api/galleries', { title: this.title })
        .then(res => {
          if (res.status === 200 && res.data.location) {
            this.picturesEndpoint = res.data
            return true
          } else {
            throw new Error('Something is fucked up')
          }
        })
        .catch(console.error)
    },
    submitPictures () {
      // return this.$refs.uploader.submit(this.picturesEndpoint)
      const endpoint = this.picturesEndpoint.location
      return new Promise((resolve, reject) => {
        this.$refs.uploader.getFiles().forEach((file, i) => {
          const formData = new FormData()

          // formData.append(name, value); // Strings
          // formData.append(name, blob, filename); // Blobs
          // formData.append(name, file, filename); // Files
          formData.set('picture', file, `${i}-${file.name}`)

          axios[endpoint.method]('/api' + endpoint.url, formData, {
            onUploadProgress (ev) {
              var percentCompleted = Math.round( (ev.loaded * 100) / ev.total )
            }
          }).then(res => {
            resolve(true)
          }).catch(reject)
        })
      })
    },
    onFilesSelected (files) {
      // validate files
    },
    fetchGalleries () {
      axios.get('/api/galleries')
      .then(res => {
        if (res.data && res.data.length) {
          this.galleries = res.data
        }
      })
    }
  },
  components: {
    PictureUploader,
    FormWizard,
    TabContent
  },
  created () {
    window.fetch('/api')
      .then(res => res.json())
      .then(body => {
        this.greeting = body.greeting
      })

    this.fetchGalleries()
  }
}
</script>

<style>
.vue-form-wizard {
  max-width: 600px;
  margin: 0 auto;
}
</style>
