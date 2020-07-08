<template>
  <div
    v-if="ready"
    class="swiper-item"
    :class="{
      'swiper-item-card': modeType === 'card',
      'animating': isAnimating,
			'touching': isTouching,
      'on-stage': onStage,
      'is-center': isCenter
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
      // isAnimating: 动态样式
      // isTouching: 动态样式
      // beforeTouchX：touch事件触发前该元素的translate值
      translate: 0,
      isAnimating: false,
      isTouching: false,
      beforeTouchX: 0,

      // modeType：模式类型，值为card表示卡片式
      // edgeCardScale：两侧卡片scale的比例
      // onStage：是否位于两侧
      modeType: "",
      edgeCardScale: 0,
      onStage: false,
      isCenter: false,

      // ready：updateItems时会对item进行位置初始化，初始化完毕之后再进行显示
      ready: false
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
      const scale =
        this.modeType === "card" && !this.isCenter ? this.edgeCardScale : 1;

      const transitionValue =
        this.isAnimating || this.isTouching
          ? `transform ${this.autoAnimDuration / 1000}s ease-in-out `
          : `none`;

      const style = {
        transform: `translateX(${this.translate}px) scale(${scale})`,
        transition: transitionValue
      };
      return autoprefixer(style);
    }
  },
  mounted() {
    this.modeType = this.$parent.modeType;
    this.edgeCardScale = this.$parent.edgeCardScale;
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

    processCardIndex(index, activeIndex) {
      const length = this.itemsCount;
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

    /**
     * 更新当前元素的transition，触发生成动态样式
     */
    updateTranslate(index, activeIndex) {
      // offsetWidth = width + 左右padding + 左右boder
      const distance = this.$parent.$el["offsetWidth"];
      return distance * (index - activeIndex);
    },

    updateCardTranslate(index, activeIndex) {
      const parentWidth = this.$parent.$el.offsetWidth;
      if (this.onStage) {
        return (
          (parentWidth *
            ((2 - this.edgeCardScale) * (index - activeIndex) + 1)) /
          4
        );
      } else if (index < activeIndex) {
        return (-(1 + this.edgeCardScale) * parentWidth) / 4;
      } else {
        return ((3 + this.edgeCardScale) * parentWidth) / 4;
      }
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
      this.ready = true;
    },

    slideTranslateCardItem(index, activeIndex) {
      // isAnimating 两种情况
      //  ● 主角
      //  ● 处于两主角中间，并且不是 itemscount - 1 => 0
      this.isAnimating = true;

      // 处理当前索引
      index = this.processCardIndex(index, activeIndex);
      this.onStage = Math.abs(index - activeIndex) <= 1;
      this.isCenter = index === activeIndex;
      this.translate = this.updateCardTranslate(index, activeIndex);

      this.ready = true;
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
.swiper-item.swiper-item-card {
  background-image: url("../../Snipaste_2020-07-07_21-39-47.png");
  border: 10px solid rgb(131, 11, 11);
  width: 50%;
}
.swiper-item.swiper-item-card.on-stage {
  z-index: 100;
}

.swiper-item.swiper-item-card.is-center {
  z-index: 102;
}
</style>
