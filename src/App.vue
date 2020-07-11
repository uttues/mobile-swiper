<template>
  <div id="app">
    <HelloSwiperCard />
    <HelloWorld />
  </div>
</template>

<script>
import HelloWorld from "./components/HelloSwiper.vue";
import HelloSwiperCard from "./components/HelloSwiperCard.vue";

export default {
  name: "App",
  components: {
    HelloWorld,
    HelloSwiperCard
  }
};
</script>

<style>
html,
body {
  height: 100%;
  overflow: auto;
  margin: 0;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
