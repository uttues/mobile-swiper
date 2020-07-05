<template>
  <div class="swiper">
    <div
      class="swiper-container"
      :style="{ height: height }"
      @touchstart="handleTouchStart"
      @touchmove="handleTouchMove"
      @touchend="handleTouchEnd"
    >
      <slot></slot>
    </div>
      <transition><button class="swiper-arrow swiper-arrow-left" @click="prev"><i>《=</i></button></transition>
      <transition><button class="swiper-arrow swiper-arrow-right" @click="next"><i>=》</i></button></transition>
    <ul class="swiper-indicators">
      <li 
        class="swiper-indicator" 
        :class="{'active': activeIndex === index}"
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
    // slideDuration: 完整轮播（滑动）一次所花时间
    // height：绝对定位，宽度撑满，高度定制
    // dragRatioMinLimit：拖拽超过这个限度就会触发轮播
    interval: {
      type: Number,
      default: 2000
    },
    slideDuration:{
      type: Number,
      default: 500
    },
    height: {
      type: String,
      default: "300px",
      required: true
    },
    // 
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
      // timer：定时器

      // 自动滑动：定时器轮播、按钮切换、指示器切换、拖拽引起的小滑动
      // autoAnimDuration：一次自动滑动所需时间（初始值由父组件传入，默认500）
      //  ● 只有在拖拽相关时需要修改，其它情况均用默认传入的slideDuration
      // isAutoSliding: 当触发的自动滑动正在进行时，拖拽无效
      
      // 拖拽事件：
      // startTouchX: touchStart
      // currentX: 
      items: [],
      initialIndex: 1, // 测试
      activeIndex: -1,
      autoAnimDuration: 0,  
      timer: null,
      isAutoSliding: false,
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
    /** 
     * 维护 isAutoSliding 状态，调用时即刻开启自动滑动状态，滑动时间后关闭
     *  ● 无需传参：直接用维护的autoAnimDuration
    */
    setAutoSlideStatus() {
      this.isAutoSliding = true
      setTimeout(()=>{
        this.isAutoSliding = false
      }, this.autoAnimDuration)
    },

    /** 
     * 维护 autoAnimDuration 状态（拖拽+尾滑完成后恢复原值slideDuration）
     * @param value 
     * @param isReset 是否需要延时重置，拖拽进行中不需要，但拖拽后引起的滑动需要重置（默认不需要）
    */
    setAutoAnimDuration(value, isReset = false) {
      const oldValue = this.autoAnimDuration
      this.autoAnimDuration = value
      if (isReset) {
        setTimeout(()=>{
          this.autoAnimDuration = this.duration
        }, oldValue)
      }
    },

    /** 
     * 轮播轨道滑动，本质上调用setActiveItem修改值（自动 开启滑动保护，setAutoSlideStatus）
     *  ● 适用情况：定时器轮播、按钮切换、指示器切换、拖拽引起的小滑动
     *  ● 注意事项：调用之前需确定自动滑动时间，autoAnimDuration
     * @param offset 正值表示：展示列表后第n张，负值表示：展示列表前第n张
    */
    playSlide(offset) {
      console.log('playSlide');
      this.setAutoSlideStatus()
      this.setActiveItem(this.activeIndex + offset)
    },
    
    /** 
     * 开启定时器
    */
    startTimer() {
      if (this.interval <= 0 || this.timer) return;
      this.timer = setInterval(this.playSlide, this.interval, 1);
    },

    /** 
     * 非自然滑动（按钮、指示器、拖拽小滑动）需要重启定时器，多数情况interval就是this.autoAnimDuration
    */
    delayRestartTimer(interval) {
      this.pauseTimer()
      setTimeout(this.startTimer, interval)
    },

    /** 
     * 关闭定时器
    */
    pauseTimer() {
      if (this.timer) {
        clearInterval(this.timer);
        this.timer = null;
      }
    },

    /**
     * 根据子组件的name属性，更新 swiper-item 组件列表items
     */
    updateItems() {
      this.items = this.$children.filter(
        child => child.$options.name === "SwiperItem"
      );
    },

    /** 
     * 调用每个swiper-item组件的 slideTranslateItem，触发元素移动
     * （负责定时器、按钮类型的轮播）
    */
    resetItemsPosition(oldIndex) {
      console.log('resetItemsPosition');
      this.items.forEach((item, index) => {
        item.slideTranslateItem(index, this.activeIndex, oldIndex);
      });
    },

    /**
     * 修改this.activeIndex的值，对边界做了处理
     */
    setActiveItem(index) {
      const oldIndex = this.activeIndex;

      if(index < 0) {
        this.activeIndex = this.items.length - 1
      } else if(index > this.items.length - 1) {
        this.activeIndex = 0;
      } else {
        this.activeIndex = index
      }
      
      // 如果index===activeIndex，不会触发watch
      if (this.activeIndex === oldIndex) {
        this.resetItemsPosition(oldIndex);
      }
    },
    

    /** 
     * 按钮滑动：前一个swiper-item
    */
    prev() {
      this.playSlide(-1)
      this.delayRestartTimer(this.autoAnimDuration)
    },
    /** 
     * 按钮滑动：下一个swiper-item
    */
    next() {
      this.playSlide(1)
      this.delayRestartTimer(this.autoAnimDuration)
    },

    /** 
     * 指示器切换滑动：点击下标为index的 swiper-item
    */
    indicatorClick(index) {
      this.playSlide(index - this.activeIndex)
      this.delayRestartTimer(this.autoAnimDuration)
    },


    /** 
     * 
    */
    handleTouchStart(event) {
      if(this.isAutoSliding) return

      this.startTouchX = event.touches[0].pageX;

      // 让子组件跟随鼠标快速移动
      this.setAutoAnimDuration(0, false)

      this.pauseTimer();

      this.items.forEach(item => {
        item.toucherStart();
      });
    },

    handleTouchMove(event) {
      if(this.isAutoSliding) return

      this.dragDistance = event.touches[0].pageX - this.startTouchX;

      this.items.forEach((item, index) => {
        item.toucherTranslateItem(index, this.activeIndex, this.dragDistance);
      });
    },

    /** 
     * Touch结束时触发，需要处理一下事情
     * 1、计算拖动的比例，判断是否需要引起轮播
     * 2、除了拖动比例为0之外，都需要引起一次 距离<width,时间<autoAnimDuration 的小滑动
     *    2.1、修改 autoAnimDuration 的值
     *    2.2、触发子组件计算位置（activeIndex变动触发||手动调用reset触发） => 用setActiveIndex
     *    2.3、定时器修改 autoAnimDuration 的值为原先组件外传入的值
     * 3、拖动结束后开启计时器
    */
    handleTouchEnd() {
      if(this.isAutoSliding) return

      const dragRatio = Math.abs(this.dragDistance / this.$el['offsetWidth'])
      let animDuration
      console.log('dragRatio:', dragRatio);
       // dragRatio>0 会触发小滑动 => 设置滑动时间，开启滑动（内部自动设置滑动保护）
      if(dragRatio === 0) {
        this.delayRestartTimer(0)
      } else if (dragRatio < this.dragRatioMinLimit) {
        
        animDuration = this.slideDuration * dragRatio
        this.setAutoAnimDuration(animDuration, true)
        this.playSlide(0)
        this.delayRestartTimer(this.autoAnimDuration)
      } else {
        // 修改滑动时间、开启滑动playSlide的顺序不能错
        animDuration = this.slideDuration * (1 - dragRatio)
        this.setAutoAnimDuration(animDuration, true)
        if(this.dragDistance > 0) {
          this.playSlide(-1)
        } else {
          this.playSlide(1)
        }
        this.delayRestartTimer(this.autoAnimDuration)
      }

      // 子组件修改自己的isTouching状态
      this.items.forEach(item => {
        item.toucherEnd();
      });
    },

  },

  mounted() {
    // 让prop初始化data，而autoAnimDuration在后期会因为各种操作而修改
    this.autoAnimDuration = this.slideDuration

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
  border: 1px solid #000;
}
.swiper-container {
  position: relative;
  overflow: hidden;
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
