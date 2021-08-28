<template>
  <div class="slider">
    <div class="slider__indicator">
      <div
          class="slider__indicator-circle"
          v-for="(chuck, index) in chunks"
          :key="index"
          :class="{ active: index === indexOfActiveChunk }"
          @click="changeActiveChunk(index)"
      >
      </div>
    </div>

    <div class="image">
      <button 
          class="btn__prev-sl"
          @click="prevSlide"
          :class="{ disable: indexOfActiveImage === 0 }"
      >
      </button>
      <img :src="activeImage" alt="current-image">
      <button 
          class="btn__next-sl"
          @click="nextSlide"
          :class="{ disable: indexOfActiveImage === chunks[indexOfActiveChunk].length - 1 }"
      >
      </button>
    </div>

    <div class="btn__load">
      <input
          id="fileUpload"
          type="file"
          hidden
          @change="onFileChange">
      <button
          class="btn__load-pk"
          @click="chooseFile">
        Загрузити з ПК
      </button>
      <button
          @click="chooseRandomFile"
          class="btn__load-st">
        Загрузити з Picsum
      </button>
    </div>

    <div class="autoplay">
      <h3 class="autoplay-title">
        Слайд шоу
      </h3>
      <label class="autoplay-box">
        <input
            class="autoplay__btn"
            type="checkbox"
            :checked="autoPlay"
            @input="autoPlay($event.target.checked)"
        >
        <span class="check round"></span>
      </label>
    </div>

    <div class="images">
      <button
          class="btn__prev-arr"
          @click="prevArray"
          :class="{ disable: indexOfActiveChunk === 0 }"
      >
      </button>
      <div class="images__box">
        <div
            class="images__item"
            v-for="(image, index) in chunks[indexOfActiveChunk]"
            :key="index"
        >
          <img
              :src="image"
              alt="image"
              @click="changeActiveImage(index)"
              :class="{ active: index === indexOfActiveImage }"
          >
          <span
              class="delete"
              ref="delete"
              :class="{ active: index === indexOfActiveImage }"
              v-if="images[0] !== activeImageNull"
              @click="deleteImage(index, indexOfActiveChunk)"
          >
          </span>
        </div>
      </div>
      <button
          class="btn__next-arr"
          @click="nextArray"
          :class="{ disable: indexOfActiveChunk === chunks.length - 1 }"
      >
      </button>
    </div>
  </div>
</template>


<script>
export default {
  name: "Slider",
  data() {
    return {
      index: 0,
      indexOfActiveImage: 0,
      activeImageNull: '/images/img-none.jpg',
      indexOfActiveChunk: 0,
      numberOfVisibleImages: 5,
      autoChange: null
    }
  },
  props: {
    images: {
      type: Array
    },
    time: {
      type: Number
    }
  },
  mounted() {
    this.autoChangeActiveImage()
  },
  computed: {
    chunks() {
      let indexArray = 0;
      let arrayLength = this.images.length;
      let tempArray = [];
      for (indexArray = 0; indexArray < arrayLength; indexArray += this.numberOfVisibleImages) {
        let myChunk = this.images.slice(indexArray, indexArray + this.numberOfVisibleImages);
        tempArray.push(myChunk);
      }
      return tempArray;
    },
    currentChunk() {
      return this.chunks[this.indexOfActiveChunk];
    },
    activeImage() {
      return this.currentChunk[this.indexOfActiveImage];
    },
  },
  methods: {
    autoPlay(value) {
     if(value) {
       this.autoChangeActiveImage()
     } else {
       clearInterval(this.autoChange)
     }
    },
    autoChangeActiveImage() {
      this.autoChange = setInterval(() => {
        //if chunk is not last
        if(this.indexOfActiveChunk !== this.chunks.length - 1) {
          //if image is not last
          if(this.indexOfActiveImage >= 0 && this.indexOfActiveImage !== this.chunks[this.indexOfActiveChunk].length - 1) {
            this.indexOfActiveImage ++
          //if image is last
          } else {
            this.indexOfActiveChunk ++
            this.indexOfActiveImage = 0
          }
        //if chunk is last
        } else {
          //if image is not last
          if(this.indexOfActiveImage !== this.chunks[this.indexOfActiveChunk].length - 1) {
            this.indexOfActiveImage ++
          //if image is last
          } else {
            this.indexOfActiveChunk = 0
            this.indexOfActiveImage = 0
          }
        }
      }, this.time)
    },
    changeActiveImage(index) {
      this.indexOfActiveImage = index;
    },
    changeActiveChunk(index) {
      this.indexOfActiveChunk = index;
      this.indexOfActiveImage = 0;
    },
    prevSlide() {
      this.indexOfActiveImage > 0 ? this.indexOfActiveImage -- : ''
    },
    nextSlide() {
      this.indexOfActiveImage < this.currentChunk.length - 1 ? this.indexOfActiveImage++ : ''
    },
    prevArray() {
      if (this.indexOfActiveChunk > 0) {
        this.indexOfActiveChunk--;
        this.indexOfActiveImage = 0;
      }
    },
    nextArray() {
      if (this.indexOfActiveChunk < this.chunks.length - 1) {
        this.indexOfActiveChunk++;
        this.indexOfActiveImage = 0;
      }
    },
    deleteImage(index, indexOfActiveChunk) {
      this.images.splice(index + (indexOfActiveChunk * this.numberOfVisibleImages), 1);
      if (index === this.indexOfActiveImage) {
        //if array end
        if (this.images.length === 0) {
          this.images.push(this.activeImageNull);
          this.indexOfActiveImage = 0;
          //if chunks end
        } else if (this.indexOfActiveChunk >= this.chunks.length) {
          this.indexOfActiveChunk--;
          this.indexOfActiveImage = 0;
          //if deleted active image on the last in the slide
        } else if (index === this.currentChunk.length) {
          this.indexOfActiveImage--;
          //
        } else {
          this.indexOfActiveImage = index;
        }
        //if deleted image before active
      } else if (index !== this.indexOfActiveImage && index < this.indexOfActiveImage) {
        this.indexOfActiveImage--;
      }
    },
    chooseFile() {
      document.getElementById("fileUpload").click()
    },
    onFileChange(e) {
      const file = e.target.files[0];
      this.url = URL.createObjectURL(file);
      this.images.push(this.url);
      this.indexOfActiveChunk = this.chunks.length - 1;
      this.indexOfActiveImage = this.currentChunk.length - 1;
      this.deleteImageNull()
    },
    async chooseRandomFile() {
      let url = 'https://picsum.photos/v2/list?page=2&limit=100'
      let response = await fetch(url);
      let data = await response.json();
      let randomNumber = Math.floor(Math.random() * data.length);
      let newImage = data[randomNumber].download_url;
      this.images.push(newImage);
      this.indexOfActiveChunk = this.chunks.length - 1;
      this.indexOfActiveImage = this.currentChunk.length - 1;
      this.deleteImageNull()
    },
    deleteImageNull() {
      if (this.images.length === 2 && this.images[0] === this.activeImageNull) {
        this.images.shift();
        this.indexOfActiveImage = 0;
      }
    }
  }
}
</script>

<style>
:root {
  --slider-bg-color: #fff;
  --slider-border-color: #dedbdb;
  --main-image-border-color: #c3bfbf;
  --general-bg-color: #c3bfbf;
  --general-border--color: #dedbdb;
  --arrow-color: #fff;
  --arrow-hover-color: #848484;
  --arrow-disable-color: #e4e4e4;
  --button-bg-hover-color: #848484;
  --button-text--color: #fff;
  --delete-btn-color: #848484;
}

.slider {
  margin: 0 auto;
  width: 960px;
  height: 740px;
  text-align: center;
  border: 5px solid var(--slider-border-color);
  background-color: var(--slider-bg-color);
}

.slider__indicator {
  height: 40px;
  width: 100%;
  background-color: var(--general-bg-color);
  border: 1px solid var(--general-border--color);
  display: flex;
  flex-direction: row;
  justify-content: center;
}

.slider__indicator-circle {
  height: 20px;
  width: 20px;
  border-radius: 50%;
  background-color: #fff;
  border: 1px solid var(--general-border--color);
  margin-top: 10px;
  cursor: pointer;
}
.slider__indicator-circle + .slider__indicator-circle {
  margin-left: 20px;
}

.slider__indicator-circle.active {
  background-color: var(--button-bg-hover-color);
  transform: scale(1.2);
}

.image {
  width: 790px;
  height: 363px;
  padding-top: 25px;
  margin: 30px auto 0;
  position: relative;
}

.image img {
  width: 726px;
  height: 100%;
  z-index: 0;
  border: 15px solid var(--main-image-border-color);
  box-sizing: border-box;
}
.images {
  background-color: #c3bfbf;
}

.images__box {
  display: flex;
  justify-content: space-around;
  width: 726px;
  margin: 0 auto 5px;
  padding: 10px 20px;
  background-color: #fff;
}

.images__item img {
  cursor: pointer;
  width: 105px;
  height: 70px;
  opacity: 0.5;
  z-index: 2;
}

.images__item + .images__item {
  margin-left: 12px;
}

.images__item {
  position: relative;
}

.images img.active {
  transform: scale(1.1);
  transition: all .5s;
  opacity: 1;
}

.btn__prev-sl,
.btn__next-sl {
  cursor: pointer;
  position: absolute;
  z-index: 1;
  top: 50%;
  transform: translate(0, -50%);
  width: 70px;
  height: 90px;
  border: 2px solid var(--general-border--color);
  background-color: var(--general-bg-color);
}

.btn__prev-sl {
  left: -105px;
}

.btn__prev-sl::before {
  position: absolute;
  left: 12px;
  content: "\f0d9";
  font-family: "Font Awesome 5 Free";
  font-weight: 900;
  color: var(--arrow-color);
  font-size: 70px;
  line-height: 70px;
}

.btn__next-sl {
  right: -105px;
}
.btn__prev-sl::before,
.btn__next-sl::after {
  top: 8px;
}

.btn__next-sl::after {
  position: absolute;
  right: 10px;
  content: "\f0da";
  font-family: "Font Awesome 5 Free";
  font-weight: 900;
  color: var(--arrow-color);
  font-size: 70px;
  line-height: 70px;
}

.btn__prev-sl:hover::before,
.btn__next-sl:hover::after,
.btn__next-arr:hover::after,
.btn__prev-arr:hover::before{
  color: var(--arrow-hover-color)
}

.btn__next-sl.disable::after,
.btn__prev-sl.disable::before,
.btn__next-arr.disable::after,
.btn__prev-arr.disable::before{
  cursor: auto;
  color: var(--arrow-disable-color);
}

.btn__prev-arr,
.btn__next-arr {
  cursor: pointer;
  right: 40px;
  background: none;
  border: none;
  position: relative;
}

.btn__next-arr {
  left: 1px;
}

.btn__next-arr::after,
.btn__prev-arr::before {
  position: absolute;
  font-family: "Font Awesome 5 Free";
  font-weight: 900;
  color: var(--arrow-color);
  font-size: 50px;
  line-height: 40px;
}

.btn__prev-arr::before {
  content: "\f100";
  right: 375px;
  top: 41px;
}

.btn__next-arr::after {
  content: "\f101";
  left: 399px;
  bottom: 46px;
}

.delete {
  position: relative;
  cursor: pointer;
  background-color: var(--general-bg-color);
  position: absolute;
  z-index: 1;
  opacity: 0;
  top: 42px;
  left: 0;
  width: 105px;
  height: 28px;
  transition: all .5s;
}

.delete::after {
  cursor: pointer;
  position: absolute;
  content: "\f2ed";
  font-family: "Font Awesome 5 Free";
  font-weight: 900;
  color: var(--delete-btn-color);
  font-size: 20px;
  line-height: 20px;
  top: 3px;
  left: 50%;
  transform: translate(-50%);
}

.delete.active {
  width: 117px;
  left: -6px;
  top: 46px;
}

.delete:hover {
  transition: all .5s;
  opacity: 1;
}

.autoplay {
  background-color: var(--general-bg-color);
  display: flex;
  justify-content: left;
  align-items: center;
  height: 47px;
}

.autoplay-title {
  margin-left: 104px;
  margin-right: 20px;
  color: var(--button-text--color);
  font-size: 22px;
  font-weight: bold;
}

.autoplay-box {
  position: relative;
  display: inline-block;
  width: 60px;
  height: 30px;
  margin: 10px 0;
}
.autoplay__btn {
  display:none;
}

.check {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #fff;
  -webkit-transition: .4s;
  transition: .4s;
}

.check:before {
  position: absolute;
  content: "";
  height: 27px;
  width: 27px;
  left: 4px;
  bottom: 1px;
  background-color: #fff;
  border: 1px solid var(--arrow-hover-color);
  -webkit-transition: .4s;
  transition: .4s;
  box-sizing: border-box;
}

input:checked + .check {
  background-color: var(--button-bg-hover-color);
}

input:checked + .check:before {
  -webkit-transform: translateX(26px);
  -ms-transform: translateX(26px);
  transform: translateX(26px);
}
.check.round {
  border-radius: 34px;
}

.check.round:before {
  border-radius: 50%;
}

.btn__load {
  display: flex;
  flex-direction: row;
  justify-content: center;
  margin-top: 15px;
  padding-bottom: 20px;
}

.btn__load-pk,
.btn__load-st {
  cursor: pointer;
  height: 40px;
  width: 481px;
  background-color: var(--general-bg-color);
  border: 2px solid var(--general-border--color);
  color: var(--button-text--color);
  font-size: 18px;
  font-weight: bold;
  margin-top: 20px;
}

.btn__load-pk:hover,
.btn__load-st:hover {
  background-color: var(--button-bg-hover-color);
}
</style>