<template>
  <div>
    <view class="avatar-wrapper">
      <div>
        <button open-type="getUserInfo" @getuserinfo="onGotUserInfo" v-if="!nickName"></button>
        <img :src="avatar" class="avatar" v-if="avatar">
        <img src="../../../static/images/avatar.svg" class="avatar" v-else>
      </div>
      <div>
        <text class="nick-name">{{nickName || '-'}}</text>
      </div>
    </view>
    <div>
      <ul>
        <li @tap="write">
          <img src="../../../static/images/write.svg" alt>写点什么
        </li>
        <li>
          <img src="../../../static/images/collect.svg" alt>我的收藏
        </li>
        <li>
          <img src="../../../static/images/concern.svg" alt>我的关注
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import Bus from "../../utils/bus.js";
export default {
  data() {
    return {
      avatar: "",
      nickName: ""
    };
  },
  methods: {
    onGotUserInfo(e) {
      if (!e.mp.detail.userInfo) return;
      const {
          userInfo: { avatarUrl, nickName }
        } = e.mp.detail,
        db = wx.cloud.database();
      wx.setStorage({
        key: "user-info",
        data: {
          avatarUrl,
          nickName
        }
      });
      this.avatar = avatarUrl;
      this.nickName = nickName;
      db
        .collection("users")
        .add({
          data: {
            avatarUrl,
            nickName
          }
        })
        .then(res => {
          console.log(res);
        });
    },
    write() {
      Bus.$emit("new", {
        type: "new"
      });
      wx.navigateTo({
        url: "/pages/write/main"
      });
    }
  },
  created() {
    const info = wx.getStorageSync("user-info");
    if (info) {
      this.avatar = info.avatarUrl;
      this.nickName = info.nickName;
    }
  }
};
</script>

<style lang="scss" scoped>
/* 0fffea */
.avatar-wrapper {
  position: relative;
  display: flex;
  padding: 32rpx;
  > div:first-of-type {
    position: relative;
  }
  > div:nth-of-type(2) {
    display: flex;
    align-items: center;
    padding-left: 15px;
  }
}
.avatar,
button {
  width: 64px;
  height: 64px;
  border-radius: 50%;
}
.avatar {
  // border: 1px solid rgba(0,0,0,0.1);
}
button {
  background: transparent;
  position: absolute;
}
ul {
  margin-top: 30px;
  li {
    font-size: 14px;
    color: #444;
    padding: 24rpx 0;
    margin: 0 32rpx;
    &:not(:last-of-type) {
      border-bottom: 1px solid #ededed;
    }
    img {
      width: 32px;
      height: 32px;
      vertical-align: middle;
      margin-right: 5px;
    }
  }
}
</style>

