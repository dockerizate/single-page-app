<template>
  <div class="picture-uploader">
    <input type="file" ref="input" style="display:none"
        :name="name" :multiple="multiple" :accept="fileTypes" @change="inputChange">

    <ul class="list-group" v-if="form.files.length">
      <file-preview v-for="file in form.files" :key="file.name"
        :file="file" :progress="progress[file.id]" @remove="removePic" />
    </ul> <!-- end previews -->

    <div class="col pu__dropper text-center p-2" id="dropzone"
      :class="{'py-5': !form.files.length}"
      ref="dropper">
      <svg viewBox="0 0 32 32" height="128" v-if="!form.files.length"
    width="128" fill="none" stroke="#CCC" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"><path d="M9 22 C0 23 1 12 9 13 6 2 23 2 22 10 32 7 32 23 23 22 M11 18 L16 14 21 18 M16 14 L16 29" /></svg>
      <p :class="form.files.length ? 'm-0 text-muted' : 'lead m-0'">
        Drag & drop files here <br> or
        <a href="#" @click.prevent
        :class="form.files.length ? 'font-weight-bold' : 'btn btn-success'"
        @click.prevent="selectFiles">
          {{ form.files.length ? 'Add more images' : 'Browse images' }}
        </a>
      </p>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import FilePreview from './FilePreview'

const sendFile = function (formData, cb) {
  const xhr = new XMLHttpRequest()
  xhr.open('POST', '/api/upload', true)
  xhr.onload = cb || xhr.onload
  xhr.send(formData);
}

export default {
  props: {
    name: String,
    accept: {
      type: Array,
      default () { return ['jpg', 'jpeg', 'png'] }
    },
    multiple: {
      type: Boolean,
      default () { return true }
    },
    maxFiles: Number,
    maxFileSize: { // in kb
      type: Number,
      default () { return 20000 }
    },
    previews: {
      type: Boolean,
      default () { return true }
    },
    previewLayout: String,
    strings: Object
  },
  data () {
    return {
      form: {
        files: []
      },
      progress: {}
    }
  },
  computed: {
    fileTypes () {
      return '.' + this.accept.join(',.')
    },
    txt () {
      return Object.assign({
        selectFiles: 'Chose files'
      }, this.strings)
    }
  },
  methods: {
    submit (endpoint) {
      return new Promise((resolve, reject) => {
        this.form.files.forEach((file, i) => {
          const formData = new FormData()

          // formData.append(name, value); // Strings
          // formData.append(name, blob, filename); // Blobs
          // formData.append(name, file, filename); // Files
          formData.set('picture', file, `${i}-${file.name}`)

          axios[endpoint.method](endpoint.location, formData, {
            uploadProgress: console.log
          }).then(resolve).catch(reject)
          // sendFile(formData, res => {
          //   console.log('res', res)
          //   if (res.status === 200) {
          //     // File(s) uploaded.
          //     console.warn('File uploaded')
          //   } else {
          //     console.error('An error occurred!')
          //   }
          // })
        })
      })
    },
    selectFiles (elem) {
      this.$refs.input.click()
    },
    inputChange (ev) {
      const files = Array.from(ev.target.files)

      this.$emit('change', files)

      files.forEach((file, index) => {
        if (!this.validateFile(file)) return

        file.id = file.name + index
        this.$set(this.progress, file.id, 0)

        if (this.previews) {
          const FR = new FileReader()

          FR.onloadend = (ev) => {
            file.src = ev.target.result
            this.form.files.push(file)
          }

          FR.readAsDataURL(file)
        }
        else {
          this.form.files.push(file)
        }
      })

    },
    validateFile (file) {
      if ((file.size / 1000) > this.maxFileSize) {
        this.$emit('max-file-size', file.id)
        return
      }
      return true
    },
    removePic (fileId) {
      const index = this.form.files.findIndex(file => file.id === fileId)
      console.log('deleting file', index)
      this.form.files.splice(index, 1)
    },
    getFiles () {
      return this.form.files
    }
  },
  components: {
    FilePreview
  },
  mounted () {
    const dropzoneId = "dropzone"

    window.addEventListener("dragenter", function(e) {
      if (e.target.id != dropzoneId) {
        e.preventDefault()
        e.dataTransfer.effectAllowed = "none"
        e.dataTransfer.dropEffect = "none"
      }
    }, false)

    window.addEventListener("dragover", function(e) {
      if (e.target.id != dropzoneId) {
        e.preventDefault()
        e.dataTransfer.effectAllowed = "none"
        e.dataTransfer.dropEffect = "none"
      }
    })

    window.addEventListener("drop", function(e) {
      if (e.target.id != dropzoneId) {
        e.preventDefault()
        e.dataTransfer.effectAllowed = "none"
        e.dataTransfer.dropEffect = "none"
      }
    })
  }
}
</script>

<style>
.pu__dropper {
  border: 3px dashed #CCC;
}
</style>
