<template>
  <div class="modal" @tap="close" v-if="showModal">
    <slot/>
  </div>
</template>

<script>
export default {
  props: ["showModal", "close"],
  methods: {}
};
</script>

<style lang="scss" scoped>
.modal {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgb(229, 245, 245);
  margin: 150rpx 40rpx 0 40rpx;
  max-height: 50%;
  border: 1px solid #0fffea;
  border-radius: 2px;
  padding: 20rpx;
  box-sizing: border-box;
  pointer-events: none;
  &::after {
    position: absolute;
    top: -10px;
    right: -10px;
    content: "x";
    width: 20px;
    height: 20px;
    line-height: 20px;
    text-align: center;
    border: 1px solid #0fffea;
    border-radius: 50%;
    z-index: 6;
    color: #fff;
    background: #0fffea;
    pointer-events: auto;
  }
}
</style>
