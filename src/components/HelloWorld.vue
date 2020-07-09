<template>
  <div class="swiper-module">
    <Swiper
      height='100px'
      :slide-duration="600"
      :interval="2000"
      show-arrow-type="always"
      handle-indication-type="clickonly"
      @change="activeIndexChange"
      :loop="true"
      :edge-card-scale="0.75"
    >
      <SwiperItem
        v-for="(item, index) in 8"
        :key="index"
      >{{index}}</SwiperItem>
      <!-- 定制箭头方法如下 -->
      <!-- 
        <template v-slot:swiper-arrow-left-slot>左箭头</template>
        <template v-slot:swiper-arrow-right-slot>右箭头</template>
      -->
    </Swiper>
    <Swiper
      height='2.58rem'
      show-arrow-type="never"
      handle-indication-type="hover"
      :drag-ratio-min-limit="0.33"
      :loop="true"
      :autoplay="true"
      style="background: red"
    >
      <SwiperItem
        v-for="(item, index) in banners"
        :key="index"
      >sss</SwiperItem>
      <!-- <SwiperItem
      v-for="(item, index) in banners"
      :key="`banner-item-${index}`"
    >
      <div class="banner-item">
        {{ index }}
      </div>
    </SwiperItem> -->

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
.swiper-module {
  border: 1px solid;
  width: 200px;
  height: 400px;
  margin: 0 auto;
}
</style>
