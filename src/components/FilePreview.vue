<template>
  <li class="media mb-2">
    <div class="file-preview mr-2" :style="{backgroundImage: `url(${file.src})`}">
    </div>
    <div class="media-body">
      <h6 class="mt-0 mb-2">
        <strong>
          {{ file.name.length > 30 ? file.name.slice(0, 20) + '...' : file.name }}
        </strong>
        <div class="float-right" v-if="progress">
          {{ progress }} %
        </div>
      </h6>
      <div class="progress" style="height:4px" v-if="progress">
        <div class="progress-bar progress-bar-striped progress-bar-animated"
          :style="{width: progress + '%'}"></div>
      </div>
      <div class="p-1">
        {{ fileSize }} Kb
      <div class="float-right">
        <a href="#" class="btn py-0 btn-sm btn-danger"
          @click.prevent="$emit('remove', file.id)">
          {{ progress ? 'cancel' : 'remove' }}
        </a>
      </div>
      </div>
    </div>
  </li>
</template>

<script>
const units = ['bytes', 'KB', 'MB', 'GB', 'TB', 'PB', 'EB', 'ZB', 'YB']

export default {
  props: {
    file: File,
    progress: Number,
    layout: {
      type: String,
      default () { return 'list' }
    }
  },
  computed: {
    fileSize () {
      let l = 0, n = parseInt(this.file.size, 10) || 0

      while(n >= 1024 && ++l) n = n/1024

      return (n.toFixed(n >= 10 || l < 1 ? 0 : 1) + ' ' + units[l])
    }
  }
}
</script>

<style scoped>
.file-preview {
  width: 64px;
  height: 64px;
  overflow: hidden;
  background-position: center center;
  background-size: cover;
}
</style>
