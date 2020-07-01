<template>
   <div class="swiper">
       <div class="swiper-container">
           <!-- <transition><button class="swiper-arrow swiper-arrow-left"><i></i></button></transition> -->
           <!-- <transition><button class="swiper-arrow swiper-arrow-right"><i></i></button></transition> -->
           <slot></slot>
       </div>
       <!-- <ul class="swiper-indicators"></ul> -->
   </div>
</template>

<script>
export default {
  name: 'Swiper',
  props: {
      // interval：自动轮播间隔时长
      // isShowIndication：
      interval: {
          type: Number,
          default: 3000
      },
  },
  data () {
      return {
          // items：swiper-item列表
          // initialIndex：暂时不接受指定，默认0，后期可以作为prop让外界决定
          // activeIndex：当前处于展示区的swiper-item索引
          // timer：定时器
          items: [],
          initialIndex: 0, 
          activeIndex: -1,
          timer: null
      }
  },
}
</script>

<style scope>
.swiper {
    position: relative;
}
.swiper-container {
    position: relative;
}
</style>
