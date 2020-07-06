<template>
    <div class="frame">
        <Swiper
            height='150px'
            :slide-duration="500"
            :interval="2000"
            show-arrow-type="hover"
            @change="activeIndexChange"
        >
            <!-- swiper-item 切换时触发change事件，回调参数：新的索引，旧的索引 -->
            <SwiperItem
                v-for="(item, index) in 8"
                :key="index"
            >{{index}}</SwiperItem>
            <!-- 定制箭头方法如下 -->
            <!-- <template v-slot:swiper-arrow-left-slot>
                左箭头
            </template>
            <template v-slot:swiper-arrow-right-slot>
                右箭头
            </template> -->
        </Swiper>
    </div>
</template>

<script>
import Swiper from "./swiper/swiper";
import SwiperItem from "./swiper/swiper-item";
export default {
    components: {
        Swiper,
        SwiperItem
    },
    data() {
        return {};
    },
    methods: {
        update() {
            console.log("hello");
        },
        activeIndexChange(newVal, oldVal) {
            console.log(`新索引：${newVal} 旧索引：${oldVal}`);
        }
    }
};
</script>

<style scoped>
.frame {
    border: 1px solid;
    width: 200px;
    height: 200px;
    margin: 0 auto;
}
</style>
