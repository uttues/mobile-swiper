<template>
  <div
    class="swiper-item"
    :class="{ 
        'animating': isAnimating,
        'touching': isTouching
        }"
    :style="itemStyle"
  >
    {{ index }}
    <slot></slot>
  </div>
</template>

<script>
import { autoprefixer } from "../../utils";
export default {
  name: "SwiperItem",
  props: {
    index: {
      type: Number
    }
  },
  data() {
    return {
      // translate: 移动距离，改变时触发生成新的样式（动态样式）
      // inStage：该元素是否处于边缘状态
      // isAnimating: 决定是否添加动作类animating
      translate: 0,
      inStage: false,
      isAnimating: false,
      isTouching: false,
      currentX: 0
    };
  },
  computed: {
    /**
     * translate的值发生改变就会自动执行，计算样式，返回一个style对象，动态样式
     */
    itemStyle() {
      const style = {
        transform: `translateX(${this.translate}px)`
      };
      return autoprefixer(style);
    }
  },
  methods: {
    processIndex(index, activeIndex, length) {
      // 当前是activeIndex是第一张，index是最后一张，返回-1，相差-1，表示二者相邻且index在左侧
      if (activeIndex === 0 && index === length - 1) {
        return -1;
      } else if (activeIndex === length - 1 && index === 0) {
        // 当前页activeIndex是最后一张，index是第一张，返回length，相差1，表示二者相邻且index在右侧
        return length;
        // 如果，index在activeIndex前一页的前面，并且之间的间隔在一半页数即以上，则返回页数长度+1，这样它们会被置于最右侧
      } else if (index < activeIndex - 1 && activeIndex - index >= length / 2) {
        return length + 1;
        // 如果，index在activeIndex后一页的后面，并且之间的间隔在一般页数即以上，则返回-2，这样它们会被置于最左侧
      } else if (index > activeIndex + 1 && index - activeIndex >= length / 2) {
        return -2;
      }
      return index;
    },
    calcTranslate(index, activeIndex) {
      // offsetWidth = width + 左右padding + 左右boder
      const distance = this.$parent.$el["offsetWidth"];
      return distance * (index - activeIndex);
    },
    /**
     * 需要轮播时，由外层 swiper 调用translateItem，触发元素移动
     * @param index 当前swiper-item索引
     * @param activeIndex 轮播需要显示在正中间的swiper-item索引
     * @param oldIndex 过去的activeIndex
     * 所有元素都会重新计算自己的translate值，生成动态样式
     * (列表中只有两个元素需要移动，旧的activeIndex（左移移走），新的activeIndex（左移移入）？？？)
     */
    timerTranslateItem(index, activeIndex, oldIndex) {
      // 列表中只有两个元素需要移动，旧的activeIndex（左移移走），新的activeIndex（左移移入）？？？
      this.isAnimating = index === activeIndex || index === oldIndex;
      const itemsCount = this.$parent.items.length;
      if (index !== activeIndex && itemsCount > 2) {
        // 处理当前索引
        index = this.processIndex(index, activeIndex, itemsCount);
      }
      // 更新当前元素active状态（是否为activeIndex），计算当前元素的Translate并修改 => 触发新的style计算，动态样式
      this.translate = this.calcTranslate(index, activeIndex);
      // this.ready = true;
    },
    toucherStart() {
      this.isAnimating = false;
      this.isTouching = true;
      this.currentX = this.translate;
      console.log("this.currentX: " + this.currentX);
    },
    toucherTranslateItem(index, activeIndex, dragDistance) {
      // 列表中只有两个元素需要移动，旧的activeIndex（左移移走），新的activeIndex（左移移入）？？？
      this.isTouching = Math.abs(index - activeIndex) <= 1;
      const itemsCount = this.$parent.items.length;
      if (index !== activeIndex && itemsCount > 2) {
        // 处理当前索引
        index = this.processIndex(index, activeIndex, itemsCount);
      }
      if (this.isTouching) {
        this.translate = this.currentX + dragDistance;
      }
    },
    toucherEnd() {
      this.isAnimating = true;
      this.isTouching = false;
    }
  },
  mounted() {
    console.log(this.$parent.$el["offsetWidth"]);
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
.animating {
  transition: transform 0.4s ease-in-out;
}
.touching {
  transition: transform 0;
}
</style>
