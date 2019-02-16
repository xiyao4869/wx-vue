<template>
  <swiper :autoplay="true" interval="2000" circular="true">
    <block v-for="(item, index) in lists" :key="index">
      <swiper-item>
        <image :src="item.src" class="slide-image" mode="aspectFill"></image>
      </swiper-item>
    </block>
  </swiper>
</template>

<script>
export default {
  props: ["lists"]
};
</script>

<style lang="scss" scoped>
.slide-image {
  width: 100%;
  height: 100%;
}
</style>
