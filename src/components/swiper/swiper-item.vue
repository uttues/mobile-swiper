<template>
   <div class="swiper-item">
       {{ testItem }}
       <slot></slot>
   </div>
</template>

<script>
export default {
    name: 'SwiperItem',
    props: {
        testItem: {
            type: String
        },
    },
    data () {
        return {
            // translate: 移动距离，改变时触发生成新的样式（动态样式）
            // inStage：该元素是否处于边缘状态
            translate: 0,
            inStage: false,
        }
    }
}
</script>

<style scope>
.swiper-item {
    position: absolute;
    width: 100%;
    height: 100px;
    background: #4962;
}
</style>
