<template>
  <div>
    <textarea auto-height placeholder="写点什么吧" v-model="desc" @input="inputValue"></textarea>
    <button @tap="submit" class="submit">提交</button>
  </div>
</template>

<script>
import Bus from "../../utils/bus.js";
export default {
  data() {
    return {
      desc: ""
    };
  },
  onShow() {
    const { index, desc } = this.$root.$mp.query;
    this.list_index = index;
    this.desc = desc;
  },
  methods: {
    submit() {
      Bus.$emit("setContent", {
        index: this.list_index,
        content: this.desc,
        change: this.$root.$mp.query.change
      });
      wx.navigateBack();
    },
    inputValue(e) {
      const { value } = e.mp.detail;
      this.desc = value;
    }
  }
};
</script>

<style lang="scss" scoped>
textarea {
  min-height: calc(100vh - 60px);
  padding: 10px;
  box-sizing: border-box;
  font-size: 16px;
}
</style>


