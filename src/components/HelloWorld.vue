<template>
  <div class="banner-swiper-container">
    <Swiper
      height='129px'
      show-arrow-type="never"
      handle-indication-type="hover"
      :drag-ratio-min-limit="0.33"
      :slide-duration="500"
      :interval="3000"
      :loop="true"
      :autoplay="true"
    >
      <SwiperItem
        v-for="(item, index) in banners"
        :key="index"
      >
        <a class="banners-item">
          <img
            :src="item.imageUrl"
            alt
          />
        </a>
      </SwiperItem>
    </Swiper>
  </div>
</template>

<script>
import Swiper from "./swiper/swiper";
import SwiperItem from "./swiper/swiper-item";
import axios from "axios";
export default {
  components: {
    Swiper,
    SwiperItem
  },
  data() {
    return {
      banners: []
    };
  },
  mounted() {
    axios.get(`http://localhost:3000/banner?type=0`).then(res => {
      console.log(res.data.banners);
      this.banners = res.data.banners;
    });
  },
  methods: {
    activeIndexChange(newVal, oldVal) {
      console.log(`新索引：${newVal} 旧索引：${oldVal}`);
    }
  }
};
</script>

<style scoped>
.banner-swiper-container {
  text-align: center;
}
.banner-swiper-container img {
  width: 343px;
  border-radius: 5px;
}
</style>
