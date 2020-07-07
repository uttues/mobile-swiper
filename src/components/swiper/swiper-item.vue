<template>
  <div
    class="swiper-item"
    :class="{
			'animating': isAnimating,
			'touching': isTouching
		}"
    :style="itemStyle"
  >
    <slot></slot>
  </div>
</template>

<script>
import { autoprefixer } from "../../utils";
export default {
  name: "SwiperItem",
  data() {
    return {
      // translate: 移动距离，改变时触发生成新的样式（动态样式）
      // isAnimating: 决定是否添加动作类animating
      // isTouching: 决定是否添加动作类touching
      // beforeTouchX：touch事件触发前该元素的translate值
      translate: 0,
      isAnimating: false,
      isTouching: false,
      beforeTouchX: 0
    };
  },
  computed: {
    /**
     * 一次轮播图片所用的时间，获取父组件中的autoAnimDuration，及时更新，默认500
     */
    autoAnimDuration() {
      return this.$parent.autoAnimDuration;
    },
    itemsCount() {
      return this.$parent.items.length;
    },

    /**
     * translate的值发生改变就会自动执行，计算样式，返回一个style对象，动态样式
     */
    itemStyle() {
      console.log(this.autoAnimDuration);

      const transitionValue =
        this.isAnimating || this.isTouching
          ? `transform ${this.autoAnimDuration / 1000}s ease-in-out`
          : `none`;
      const style = {
        transform: `translateX(${this.translate}px)`,
        transition: transitionValue
      };
      return autoprefixer(style);
    }
  },
  methods: {
    /**
     * 返回当前元素处理后的index值，用于计算新的translate值
     */
    processIndex(index, activeIndex) {
      const length = this.itemsCount;
      if (activeIndex === 0 && index === length - 1) {
        return -1;
      } else if (activeIndex === length - 1 && index === 0) {
        return length;
      }
      return index;
    },

    /**
     * 更新当前元素的transition，触发生成动态样式
     */
    updateTranslate(index, activeIndex) {
      // offsetWidth = width + 左右padding + 左右boder
      const distance = this.$parent.$el["offsetWidth"];
      return distance * (index - activeIndex);
    },

    /**
     * 需要滑动时，由外层 swiper 调用slideTranslateItem，触发元素移动
     * @param index 当前swiper-item索引
     * @param activeIndex 轮播需要显示在正中间的swiper-item索引
     * @param oldIndex 过去的activeIndex
     * 所有元素都会重新计算自己的translate值，生成动态样式
     * (列表中只有两个元素需要移动，旧的activeIndex（左移移走），新的activeIndex（左移移入）？？？)
     */
    slideTranslateItem(index, activeIndex, oldIndex) {
      // isAnimating 两种情况
      //  ● 主角
      //  ● 处于两主角中间，并且不是 itemscount - 1 => 0
      this.isAnimating =
        index === oldIndex ||
        index === activeIndex ||
        (((oldIndex < index && index < activeIndex) ||
          (activeIndex < index && index < oldIndex)) &&
          (oldIndex !== this.itemsCount - 1 || activeIndex !== 0) &&
          (oldIndex !== 0 || activeIndex !== this.itemsCount - 1));

      // 处理当前索引
      index = this.processIndex(index, activeIndex, this.itemsCount);

      // 计算当前元素的Translate并修改 => 触发新的style计算，动态样式
      this.translate = this.updateTranslate(index, activeIndex);
      // this.ready = true;
    },

    /**
     *  处理元素下标，更新translate（实现元素拖拽跟随）
     */
    toucherTranslateItem(index, activeIndex, dragDistance) {
      // 如果不执行这一个processIndex，则不会实现循环播放
      index = this.processIndex(index, activeIndex);

      if (this.isTouching) {
        this.translate = this.beforeTouchX + dragDistance;
      }
    },

    /**
     * 修改元素动态类（状态）
     */
    toucherStart(index, activeIndex) {
      this.isAnimating = false;
      // 相邻的 Math.abs(index - activeIndex) <= 1 或者 同为列表边界
      this.isTouching = Math.abs(index - activeIndex) <= 1;
      this.beforeTouchX = this.translate;
    },

    /**
     * 修改元素动态类（状态），在父组件处理touchEnd时会重新开启计时器（轮播），所以这边无需处理isAnimating
     */
    toucherEnd() {
      this.isTouching = false;
    }
  }
};
</script>

<style scope>
.swiper-item {
  position: absolute;
  width: 100%;
  height: 100%;
  background: #4962;
}
.touching {
  transition: transform 0;
}
</style>
