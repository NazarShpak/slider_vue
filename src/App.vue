<template>
  <div class="container">
    <div id="app container">
      <Slider
        :images="images"
        :time="time"
      >
      </Slider>
    </div>
  </div>
</template>

<script>

import Slider from './components/Slider.vue'

export default {
  name: 'App',
  data() {
    return {
      time: 3000,
      images: [
        '/images/img-1.jpg',
        '/images/img-2.jpg',
        '/images/img-3.jpg',
        '/images/img-4.jpg',
        '/images/img-5.jpg',
        '/images/img-6.jpg',
        '/images/img-7.jpg',
        '/images/img-8.jpg',
        '/images/img-9.jpg',
        '/images/img-10.jpg',
        '/images/img-11.jpg',
        '/images/img-12.jpg',
        '/images/img-13.jpg',
        '/images/img-14.jpg',
        '/images/img-15.jpg',
        '/images/img-16.jpg',
        '/images/img-17.jpg',
        '/images/img-18.jpg',
        '/images/img-19.jpg',
        '/images/img-20.jpg',
        '/images/img-21.jpg',
        '/images/img-22.jpg',
      ]
    }
  },
  components: {
    Slider,
  }
}
</script>

<style>
.container {
  margin: 0 auto;
  padding: 0 10px;
}
</style>
