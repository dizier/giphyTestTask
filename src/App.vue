<template>
  <div id="app">
    <div id="header">
      <div id="logo">LOGO</div>
      <input @input="onInput" maxlength="150">
    </div>
    <div id="wrapper" @scroll="onScroll">
      <div class="loader" v-if="isLoaderShown">
        <div id="fountainG">
          <div id="fountainG_1" class="fountainG"></div>
          <div id="fountainG_2" class="fountainG"></div>
          <div id="fountainG_3" class="fountainG"></div>
          <div id="fountainG_4" class="fountainG"></div>
          <div id="fountainG_5" class="fountainG"></div>
          <div id="fountainG_6" class="fountainG"></div>
          <div id="fountainG_7" class="fountainG"></div>
          <div id="fountainG_8" class="fountainG"></div>
        </div>
      </div>
      <div id="images" v-else>
        <app-image-gif
          v-for="image, index in images"
          :key="index"
          :title="image.title"
          :url_image="image.url"
          :img_width="image.img_width"
          :img_height="image.img_height"
          :img_back="image.img_back">
        </app-image-gif>
      </div>
    </div>
  </div>
</template>

<script>
import imageGif from "./ImageGif";

export default {
  name: 'app',
  data() {
    return {
      init_images: [],
      images: [],
      API_KEY: 'e0d4azkNIfPm9vqL0ZO7L3map9pTVV3C',
      resource_url: 'http://api.giphy.com/v1/gifs',
      loading: false,
      isLoaderShown: false,
      last_query:'',
      images_per_num: 100,
      init_images_num: 45,
      pages_num: 0,
      offset:0,
      lastSearchEvent:null
    }
  },
  methods: {
    loadData: function (query) {
      this.loading = true
      return this.$http.get(query).then(
        function (response) {
          this.pages_num = Math.ceil(response.data.pagination.total_count/this.images_per_num) - 1
          for(let item of response.data.data) {
            this.images.push(this.createImageVO(item))
          }
          this.loading = false
          return this.images.length === 0
        },
        function (reject) {
          console.log(reject);
          this.loading = false
        }
      )
    },
    onInput: function(event) {
      event.target.addEventListener("keyup", this.onSearch)
    },
    onSearch: function (event) {
      event.target.removeEventListener("keyup", this.onSearch)
      this.offset = 0
      this.images.splice(0)

      if(event.target.value === '') {
        this.images = this.images.concat(this.init_images)
        return
      }

      this.isLoaderShown = true
      this.last_query = `${this.resource_url}/search?q=${event.target.value}&api_key=${this.API_KEY}&limit=${this.images_per_num}`
      this.loadData(this.last_query + `&offset=${this.offset}`).then(notFound => {
        if(notFound) {
          this.getRandomForNotFound()
        } else {
          this.isLoaderShown = false
        }
      })
    },
    onScroll: function (event) {
      if(this.offset < this.pages_num) {
        let block = event.target
        let images = block.firstElementChild
        if (images.offsetHeight - block.offsetHeight <= block.scrollTop && !this.loading) {
          this.offset += this.images_per_num;
          this.loadData(this.last_query + `&offset=${this.offset}`);
        }
      }
    },
    getRandColor: function () {
      return Math.round(Math.random()*255)
    },
    createImageVO: function (item) {
      return {
        id: item.id,
        title: item.title,
        url: item.images.fixed_height_small.url,
        img_width: item.images.fixed_height_small.width,
        img_height: item.images.fixed_height_small.height,
        img_back: `rgb(${this.getRandColor()}, ${this.getRandColor()}, ${this.getRandColor()}`
      }
    },
    getRandomForNotFound: function () {
      this.loading = true
      this.$http.get(`${this.resource_url}/search?api_key=${this.API_KEY}&q=not found`).then(
        function (response) {
          this.images.splice(0)
          this.images.push(this.createImageVO(response.data.data[Math.round(Math.random()*(response.data.data.length -1))]))
          this.loading = false
        },
        function (reject) {
          console.log(reject);
          this.loading = false
        }
      ).then(() => {
        this.isLoaderShown = false
      })
    }
  },
  components: {
    appImageGif: imageGif
  },
  mounted: function() {
    this.loading = true
    this.isLoaderShown = true
    let counterQuery = this.init_images_num
    let counterResponse = 0
    while(counterQuery-- > 0) {
      this.$http.get(`${this.resource_url}/random?api_key=${this.API_KEY}&tag=&rating=G`).then(
        function (response) {
          this.init_images.push(this.createImageVO(response.data.data))
          this.loading = false
        },
        function (reject) {
          console.log(reject)
          this.loading = false
        }
      ).then(() => {
        if(++counterResponse === this.init_images_num) {
          this.images = this.images.concat(this.init_images)
          this.isLoaderShown = false
        }
      })
    }
  }
}
</script>
