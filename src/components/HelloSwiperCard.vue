<template>
  <div class="banner-swiper-container">
    <Swiper
      height='300px'
      show-arrow-type="never"
      handle-indication-type="hover"
      :drag-ratio-min-limit="0.5"
      :loop="true"
      :autoplay="true"
      mode-type="card"
    >
      <SwiperItem
        v-for="(item, index) in swiperSongLists"
        :key="index"
      >
        <a class="banner-item">
          <SongListItem
            class="songlist-card"
            :key="item.id"
            :playCount="item.playCount"
            :name="item.name"
            :imgUrl="item.coverImgUrl"
            size="100%"
          />
        </a>
      </SwiperItem>
    </Swiper>
  </div>
</template>

<script>
import axios from "axios";
import SongListItem from "./square-card";
import { Swiper, SwiperItem } from "./swiper-rotate";
export default {
  props: {
    list: {
      type: Array,
      default: () => []
    }
  },
  data() {
    return {
      swiperSongLists: []
    };
  },
  components: {
    Swiper,
    SwiperItem,
    SongListItem
  },
  mounted() {
    axios.get(`http://10.255.206.170:3000/top/playlist`).then(res => {
      console.log(res.data.playlists);
      this.swiperSongLists = res.data.playlists.slice(0, 3);
    });
  }
};
</script>

<style scoped>
.banner-swiper-container {
  box-sizing: border-box;
  width: 375px;
  padding: 0 17px;
}
</style>