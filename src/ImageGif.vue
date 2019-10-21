<template>
  <div :style="{'width': img_width + 'px', 'height': img_height + 'px', 'background-color': img_back}">
    <img :src="url_image" :title="title"/>
  </div>
</template>

<script>
  export default {
    name: 'ImageGif',
    props: ['title', 'url_image', 'img_width', 'img_height', 'img_back']
  }
</script>
