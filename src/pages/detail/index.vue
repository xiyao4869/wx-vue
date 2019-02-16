<template>
  <div>
    <h2>{{title}}</h2>
    <div class="cate-avatar">
      <span>{{cate_name}}</span>
      <span class="avatar-name">
        <img :src="avatarUrl" alt class="avatar">
        {{nickName}}
      </span>
    </div>
    <div v-for="(se,i) in section" :key="i" class="section">
      <img :src="se.fileID" alt>
      <p>{{se.desc}}</p>
    </div>
  </div>
</template>

<script>
const app = getApp();
export default {
  data() {
    return {
      title: "",
      cate_name: "",
      avatarUrl: "",
      nickName: "",
      section: []
    };
  },
  onReady() {
    const {
      article: {
        cate_name,
        title,
        section,
        auth: { avatarUrl, nickName }
      }
    } = app.globalData;
    this.section = section;
    this.cate_name = cate_name;
    this.avatarUrl = avatarUrl;
    this.nickName = nickName;
    this.title = title;
    wx.setNavigationBarTitle({
      title
    });
  }
};
</script>

<style lang="scss" scoped>
h2 {
  text-align: center;
  padding-top: 10px;
}
.avatar-name {
  float: right;
}
.cate-avatar {
  font-size: 12px;
  padding: 15px;
  color: #999;
}
.avatar {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  vertical-align: middle;
}
.section {
  margin: 0 15px;
  img {
    width: 100%;
  }
}
p {
  font-size: 14px;
  padding: 10px 0;
}
</style>


