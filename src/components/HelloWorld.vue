<template>
  <div class="frame">
    <Swiper height='150px' :slide-duration="500" :interval="2000">
      <SwiperItem
        v-for="(item, index) in 8"
        :key="index"
      >{{index}}</SwiperItem>
    </Swiper>
  </div>
</template>

<script>
import Swiper from "./swiper/swiper";
import SwiperItem from "./swiper/swiper-item";
export default {
  components: {
    Swiper,
    SwiperItem
  },
  data() {
    return {};
  },
  methods: {
    update() {
      console.log("hello");
    }
  }
};
</script>

<style scoped>
.frame {
  border: 1px solid;
  width: 200px;
  height: 200px;
  margin: 0 auto;
}
</style>
