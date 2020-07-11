<!-- 展示歌单item，可传入size来控制大小 -->
<template>
  <div
    class="square-card-wrapper"
    :style="{'width': size}"
  >
    <img
      :style="{'width': size, 'height': size}"
      :src="imgUrl"
      alt
    />
    <p :style="{'width': size}">{{ name }}</p>
    <span class="play-count">
      <i class="iconfont icon-bofang" />
      <span>{{ playCount | showCountingUnit }}</span>
    </span>
  </div>
</template>

<script>
export default {
  props: {
    playCount: Number,
    name: String,
    imgUrl: String,
    size: {
      type: String,
      default: "1.5rem"
    }
  }
};
</script>

<style scoped>
.square-card-wrapper {
  position: relative;
  cursor: pointer;
}
img {
  box-sizing: border-box;
  vertical-align: bottom;
  border: 0.01rem solid rgb(238, 238, 238, 0.7);
  border-radius: 0.1rem;
}
p {
  margin-top: 0.05rem;
  font-size: 0.24rem;
  color: #333333;

  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
.play-count {
  position: absolute;
  top: 0.12rem;
  right: 0.14rem;
  font-size: 0.4rem;
  color: #fff;
  transform-origin: top right;
  transform: scale(0.5);
}
.iconfont.icon-bofang {
  margin-right: 0.02rem;
  font-size: 0.4rem;
  transform: scale(0.8);
  display: inline-block;
}
</style>
