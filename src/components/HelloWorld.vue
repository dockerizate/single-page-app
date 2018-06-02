<template>
  <div class="hello">
    <h1>{{ greeting }}</h1>

    <picture-uploader />

    <template v-if="files.length">
      <div v-for="img in files">
        <img :src="img.src" width="250">
      </div>
    </template>
  </div>
</template>

<script>
import PictureUploader from './PictureUploader'

export default {
  name: 'HelloWorld',
  data () {
    return {
      greeting: 'Fetching from /api...',
      files: [
        {src: 'http://dev.local:9000/app-pics/vvv.jpg'}
      ]
    }
  },
  components: {
    PictureUploader
  },
  created () {
    window.fetch('/api')
      .then(res => res.json())
      .then(body => {
        this.greeting = body.greeting
      })
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
