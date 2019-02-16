<script>
export default {
  created() {
    if (!wx.cloud) {
      console.error("请使用 2.2.3 或以上的基础库以使用云能力");
    } else {
      console.log("use cloud");
      wx.cloud.init({
        traceUser: true
      });
    }
  }
};
</script>

<style lang="scss">
@import "../static/style/font.wxss";
button.submit {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  border-radius: 0;
  background: linear-gradient(to right, #7ae9e0, #ccffff);
  color: #fff;
  &::after {
    border: none;
  }
}
</style>

