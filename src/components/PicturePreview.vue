<template>
  <div>
    <img :src="opts.src">
  </div>
</template>

<script>
export default {
  props: ['opts']
}
</script>

<style scoped>
img {
  max-width: 250px;
}
</style>
