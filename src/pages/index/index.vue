<template>
  <div>
    <swiper :lists="imgUrls"/>
    <div class="article-title">随便看看</div>
    <div
      v-for="(article,index) in articles"
      :key="index"
      class="arctice"
      @tap="goDeatil"
      :data-article="article"
    >
      <img :src="article.surface_img" alt v-if="article.surface_img" class="surface-img">
      <div>
        <span>{{article.title}}</span>
        <div class="info">
          <span class="cate-name">{{article.cate_name}}</span>
          <span class="auth" v-if="article.auth">
            <img :src="article.auth.avatarUrl" alt>
            {{article.auth.nickName}}
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import swiper from "@/components/swiper";
import Vue from "vue";
const app = getApp();
export default {
  data() {
    return {
      imgUrls: [],
      articles: []
    };
  },

  components: {
    swiper
  },

  methods: {
    getBanner() {
      const db = wx.cloud.database(),
        banners = db.collection("bannerList");
      banners.get().then(res => {
        this.imgUrls = res.data;
      });
    },
    getList() {
      const db = wx.cloud.database(),
        articles = db.collection("articles"),
        categroys = db.collection("categroy"),
        users = db.collection("users");
      articles.get().then(res => {
        const articles = res.data;
        articles.map(item => {
          item.surface_img = item.section[0].fileID;
          categroys.get().then(res => {
            res.data.map(ele => {
              ele.children.map(c => {
                if (item.cate_id === c.cate_id) {
                  item.cate_name = c.value;
                }
              });
            });
          });
          users
            .where({
              _openid: item._openid
            })
            .get()
            .then(res => {
              let { avatarUrl = "", nickName = "" } = res.data[0] || {};
              // 给对象添加响应式属性
              Vue.set(item, "auth", {
                avatarUrl,
                nickName
              });
              this.articles = articles;
            });
        });
      });
    },
    goDeatil(e) {
      const { article } = e.currentTarget.dataset;
      app.globalData.article = article;
      wx.navigateTo({
        url: `/pages/detail/main`
      });
    }
  },

  created() {
    this.getBanner();
  },
  onShow() {
    this.getList();
  }
};
</script>

<style lang="scss" scoped>
.article-title {
  position: relative;
  font-size: 14px;
  margin: 32rpx 0 0 32rpx;
  &::before {
    position: absolute;
    content: "";
    width: 6px;
    height: 100%;
    left: -10px;
    border-radius: 2px;
    background: linear-gradient(to right, #7ae9e0, #ccffff);
  }
}
.surface-img {
  width: 64px;
  height: 64px;
  margin-right: 10px;
}
.arctice {
  display: flex;
  font-size: 14px;
  margin: 0 16px;
  padding: 15px 0;
  &:not(:last-of-type) {
    border-bottom: 1px solid #ededed;
  }
  > div {
    flex: 1;
    display: flex;
    justify-content: space-between;
    flex-direction: column;
    > span {
      font-weight: 500;
    }
  }
  .info {
    font-size: 12px;
    height: 32px;
    line-height: 32px;
  }
  .cate-name {
    color: #7ae9e0;
  }
  .auth {
    float: right;
    color: #999;
    img {
      width: 20px;
      height: 20px;
      border-radius: 50%;
      vertical-align: middle;
    }
  }
}
</style>
