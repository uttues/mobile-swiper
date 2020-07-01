<template>
    <div class="frame">
        <Swiper>
            <SwiperItem v-for="(item, index) in 8"
                        :key="index"
                        :index="index" />
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
        }
    }
};
</script>

<style scoped>
.frame {
    border: 1px solid;
    width: 120px;
    height: 200px;
    margin: 0 auto;
}
</style>
