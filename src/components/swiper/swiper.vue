<template>
  <div class="swiper">
    <div
      class="swiper-container"
      :style="{ height: height }"
      @touchstart="handleTouchStart"
      @touchmove="handleTouchMove"
      @touchend="handleTouchEnd"
    >
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
      default: 2000
    },
    animDuration:{
      type: Number,
      default: 500
    },
    height: {
      type: String,
      default: "300px",
      required: true
    }
  },
  data() {
    return {
      // items：swiper-item列表
      // initialIndex：暂时不接受指定，默认0，后期可以作为prop让外界决定
      // activeIndex：当前处于展示区的swiper-item索引
      // timer：定时器
      // isTimerSliding: 当定时器引起的滑动正在进行时，拖拽无效
      
      // 拖拽事件：
      // startTouchX: touchStart
      // currentX: 
      items: [],
      initialIndex: 4, // 测试
      activeIndex: -1,
      timer: null,
      startTouchX: 0,
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
    pauseTimer() {
      if (this.timer) {
        clearInterval(this.timer);
        this.timer = null;
      }
    },
    updateItems() {
      this.items = this.$children.filter(
        child => child.$options.name === "SwiperItem"
      );
    },
    resetItemsPosition(oldIndex) {
      this.items.forEach((item, index) => {
        item.timerTranslateItem(index, this.activeIndex, oldIndex);
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
    },

    handleTouchStart(event) {
      this.startTouchX = event.touches[0].pageX;
      console.log("handleTouchStart", this.startTouchX);
      this.pauseTimer();
      console.log("start");

      this.items.forEach(item => {
        item.toucherStart();
      });
    },

    handleTouchMove(event) {
      let dragDistance = event.touches[0].pageX - this.startTouchX;
      console.log(dragDistance);

      this.items.forEach((item, index) => {
        item.toucherTranslateItem(index, this.activeIndex, dragDistance);
      });
    },

    handleTouchEnd() {
      this.startTimer();

      this.items.forEach(item => {
        item.toucherEnd();
      });
    }
  },

  mounted() {
    this.updateItems();
    this.$nextTick(() => {
      this.startTimer();
    });
  },
  beforeDestroy() {
    this.pauseTimer();
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
  overflow: hidden;
  height: 100%;
}
</style>
