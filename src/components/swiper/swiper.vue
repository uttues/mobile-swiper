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
  name: "Swiper",
  props: {
    // interval：自动轮播间隔时长
    interval: {
      type: Number,
      default: 1000
    }
  },
  data() {
    return {
      // items：swiper-item列表
      // initialIndex：暂时不接受指定，默认0，后期可以作为prop让外界决定
      // activeIndex：当前处于展示区的swiper-item索引
      // timer：定时器
      items: [],
      initialIndex: 0,
      activeIndex: -1,
      timer: null
    };
  },
  watch: {
    items() {
      this.setActiveItem(this.initialIndex);
    },
    activeIndex(val, oldVal) {
      this.resetItemsPosition(oldVal);
    }
  },
  mounted() {
    this.updateItems();
    this.$nextTick(() => {
      this.startTimer();
    });
  },
  methods: {
    playSlide() {
      if (this.activeIndex === this.items.length - 1) {
        this.activeIndex = 0;
      } else {
        this.activeIndex++;
      }
    },
    startTimer() {
      if (this.interval <= 0 || this.timer) return;
      this.timer = setInterval(this.playSlide, this.interval);
    },
    updateItems() {
      this.items = this.$children.filter(
        child => child.$options.name === "SwiperItem"
      );
    },
    resetItemsPosition(oldIndex) {
      this.items.forEach((item, index) => {
        item.translateItem(index, this.activeIndex, oldIndex);
      });
    },
    /**
     * 修改this.activeIndex的值
     */
    setActiveItem(index) {
      const oldIndex = this.activeIndex;
      this.activeIndex = index;
      // 如果index===activeIndex，不会触发watch
      if (this.activeIndex === oldIndex) {
        this.resetItemsPosition(oldIndex);
      }
    },

    prev() {
      this.setActiveItem(this.activeIndex - 1);
    },
    next() {
      this.setActiveItem(this.activeIndex + 1);
    }
  }
};
</script>

<style scope>
.swiper {
  position: relative;
  overflow: hidden;
}
.swiper-container {
  position: relative;
  height: 100%;
}
</style>
