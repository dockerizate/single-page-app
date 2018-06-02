<template>
  <div>
    <ul id="preview-pictures">
      <li v-for="pic in form.files" :key="pic.title">
        <picture-preview :opts="pic" @remove="removePic" />
      </li>
    </ul>
    <form action="" @submit.prevent="submit">
      <input type="file" name="pictures" multiple @change="inputChange">

      <button type="submit">Submit</button>
    </form>
  </div>
</template>

<script>
import PicturePreview from './PicturePreview'

const isValidImage = function (file) {
  return file.type.match('image.*')
}

const sendFiles = function (formData, cb) {
  const xhr = new XMLHttpRequest()
  xhr.open('POST', '/api/upload', true)
  xhr.onload = cb || xhr.onload
  xhr.send(formData);
}

export default {
  data () {
    return {
      form: {
        files: []
      }
    }
  },
  methods: {
    submit (ev) {
      this.form.files.forEach((file, i) => {
        const formData = new FormData()

        // formData.append(name, value); // Strings
        // formData.append(name, blob, filename); // Blobs
        // formData.append(name, file, filename); // Files
        formData.append('picture', file, `${i}-${file.name}`);

        sendFiles(formData, res => {
          console.log('res', res)
          if (res.status === 200) {
            // File(s) uploaded.
            console.warn('File uploaded')
          } else {
            console.error('An error occurred!')
          }
        })
      })
    },
    inputChange (ev) {
      this.form.files = []
      Array.from(ev.target.files).forEach(file => {
        const FR = new FileReader()

        FR.onloadend = (ev) => {
          file.src = ev.target.result

          if (isValidImage(file)) {
            this.form.files.push(file)
          }
          else {
            throw new Error('Not an image')
          }
        }

        FR.readAsDataURL(file)
      })
    },
    removePic (index) {
      this.form.files.splice(index, 1)
    }
  },
  components: {
    PicturePreview
  }
}
</script>

<style>
</style>
