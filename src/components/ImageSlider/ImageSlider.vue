<template>
  <carousel :items-to-show="1" :wrapAround="true">
    <slide v-for="(image, index) in images" :key="`slide-${index}`">
      <img :src="image.Url" alt="image" @error="imageLoadOnError" />
    </slide>
    <template #addons>
      <navigation />
      <pagination />
    </template>
  </carousel>
</template>

<script>
import "vue3-carousel/dist/carousel.css";
import { Carousel, Slide, Pagination, Navigation } from "vue3-carousel";

export default {
  name: "App",
  components: {
    Carousel,
    Slide,
    Pagination,
    Navigation,
  },
  props: {
    images: Array,
  },
  methods: {
    imageLoadOnError(e) {
      e.target.src = require("../../assets/no-image.png");
    },
  },
};
</script>

<style lang="scss">
.carousel__pagination {
  position: relative;
  display: flex;
  justify-content: center;
  list-style: none;
  line-height: 0;
  margin: -40px 0 30px 0;
  z-index: 99999;
  padding-left: 0;
  .carousel__pagination-button {
    &:after {
      width: 12px;
      height: 12px;
      border-radius: 50%;
    }
  }
}
img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-top-left-radius: 8px;
  border-top-right-radius: 8px;
}
</style>
