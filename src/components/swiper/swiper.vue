<template>
  <div class="swiper">
    <div
      class="swiper-container"
      :style="{ height: height }"
      @touchstart="handleTouchStart"
      @touchmove="handleTouchMove"
      @touchend="handleTouchEnd"
    >
      <transition><button class="swiper-arrow swiper-arrow-left" @click="prev"><i>《=</i></button></transition>
     <transition><button class="swiper-arrow swiper-arrow-right" @click="next"><i>=》</i></button></transition>
      <slot></slot>
    </div>
    <ul class="swiper-indicators">
      <li class="swiper-indicator" 
        :class="{'active': activeIndex===index}"
        v-for="(item, index) in items" 
        :key="index" 
        @click="indicatorClick(index)">
      </li>
    </ul>
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
    duration:{
      type: Number,
      default: 500
    },
    height: {
      type: String,
      default: "300px",
      required: true
    },
    // 超过这个限度就可以触发轮播
    dragRatioMinLimit:{
      type:Number,
      default:0.25
    }
  },
  data() {
    return {
      // items：swiper-item列表
      // initialIndex：暂时不接受指定，默认0，后期可以作为prop让外界决定
      // activeIndex：当前处于展示区的swiper-item索引
      // animDuration：切换一次所需时间（为了修改父组件传入的值）
      // timer：定时器
      // isTimerSliding: 当定时器引起的滑动正在进行时，拖拽无效
      
      // 拖拽事件：
      // startTouchX: touchStart
      // currentX: 
      items: [],
      initialIndex: 4, // 测试
      activeIndex: -1,
      animDuration: 0,  //
      timer: null,
      isTimerSliding: false,
      startTouchX: 0,
      dragDistance: 0
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
    setCompleteSlideStatus(animDuration = this.duration) {
      this.isTimerSliding = true
      setTimeout(()=>{
        this.isTimerSliding = false
      }, animDuration)
    },
    playSlide() {
      this.setCompleteSlideStatus()
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

      if(index < 0) {
        this.activeIndex = this.items.length - 1
      } else if(index >= this.items.length) {
        this.activeIndex = 0;
      } else {
        this.activeIndex = index
      }
      
      // 如果index===activeIndex，不会触发watch
      if (this.activeIndex === oldIndex) {
        this.resetItemsPosition(oldIndex);
      }
    },

    prev() {
      this.setCompleteSlideStatus()
      this.setActiveItem(this.activeIndex - 1);
    },
    next() {
      this.setCompleteSlideStatus()
      this.setActiveItem(this.activeIndex + 1);
    },
    indicatorClick(index) {
      this.setCompleteSlideStatus()
      this.setActiveItem(index);
    },

    handleTouchStart(event) {
      // console.log('this.isTimerSliding:' + this.isTimerSliding);
      // console.log('handleTouchStart');
      if(this.isTimerSliding) return

      this.startTouchX = event.touches[0].pageX;

      // 让子组件跟随鼠标快速移动
      this.animDuration = 0

      this.pauseTimer();

      this.items.forEach(item => {
        item.toucherStart();
      });
    },

    handleTouchMove(event) {
      if(this.isTimerSliding) return

      this.dragDistance = event.touches[0].pageX - this.startTouchX;

      this.items.forEach((item, index) => {
        item.toucherTranslateItem(index, this.activeIndex, this.dragDistance);
      });
    },

    /** 
     * Touch结束时触发，需要处理一下事情
     * 1、计算拖动的比例，判断是否需要引起轮播
     * 2、除了拖动比例为0之外，都需要引起一次 距离<width,时间<animDuration 的小滑动
     *    2.1、修改 animDuration 的值
     *    2.2、触发子组件计算位置（activeIndex变动触发||手动调用reset触发） => 用setActiveIndex
     *    2.3、定时器修改 animDuration 的值为原先组件外传入的值
     * 3、拖动结束后开启计时器
    */
    handleTouchEnd() {
      if(this.isTimerSliding) return

      const dragRatio = Math.abs(this.dragDistance / this.$el['offsetWidth'])

      // 不论dragRatio是否可以触发轮播（但是一定会有滑动，所以需要短暂的改变animDuration的值）
      if(dragRatio >= this.dragRatioMinLimit) {
        const newAnimDuration = this.duration * (1 - dragRatio)
        
        if(this.dragDistance > 0) {
          // 顺序不能错
          this.animDuration = newAnimDuration
          
          this.setActiveItem(this.activeIndex - 1)
        } else {
          this.animDuration = newAnimDuration
          
          this.setActiveItem(this.activeIndex + 1)
        }
        
        setTimeout(() => {
          this.animDuration = this.duration
          this.startTimer()
        }, newAnimDuration)

      } else if(dragRatio > 0) {
        const newAnimDuration = this.duration * dragRatio
        this.animDuration = newAnimDuration
        this.setActiveItem(this.activeIndex)

        setTimeout(() => {
          this.animDuration = this.duration
          this.startTimer()
        }, newAnimDuration)

      } else {
        this.startTimer()
      }


      // 子组件修改自己的isTouching状态
      this.items.forEach(item => {
        item.toucherEnd();
      });
    }
  },

  mounted() {
    // 让prop初始化data，以便于修改数据
    this.animDuration = this.duration
    console.log(this.animDuration);

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
  border: 1px solid #000;
}
.swiper-container {
  position: relative;
  /* overflow: hidden; */
  height: 100%;
}
.swiper-indicator {
  display: inline-block;
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background-color: #f00;
}
.swiper-indicator.active {
  background-color: #0f0;
}
</style>
