<template>
    <div class="frame">
        <Swiper>
            <SwiperItem v-for="(item, index) in imgLists" :key="index" :test-item="item" />
        </Swiper>
    </div>
</template>

<script>
import Swiper from './swiper/swiper'
import SwiperItem from './swiper/swiper-item'
export default {
    components: {
        Swiper,
        SwiperItem
    },
    data () {
        return {
            imgLists: [1, 2, 3]
        }
    },
  methods: {
      update () {
          console.log('hello');
      }
  }
};
</script>

<style scoped>
.frame {
    border: 1px solid;
    width: 120px;
    height: 200px;
    margin: 0 auto;
}
</style>
