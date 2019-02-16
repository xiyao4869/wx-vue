<template>
  <div class="wrapper">
    <div class="categroy">
      分类:
      <span @tap="toggleModal" class="flex-item">{{selected_cate || '请选择' }}</span>
    </div>
    <div class="title">
      标题:
      <input
        class="flex-item"
        @input="bindKeyInput"
        placeholder="请输入标题"
        v-if="!show"
        v-model="title"
      >
    </div>
    <div class="imgs-wrapper">
      <img src="../../../static/images/upload_img.svg" alt @tap="uploadImg" data-index="0">
      <img src="../../../static/images/text.svg" alt @tap="addDesc" data-index="0">
    </div>
    <div class="section-list">
      <div v-for="(item,index) in sections" :key="index">
        <!-- :src="item.img_url || img_holder" 这么写真机不显示 -->
        <div class="section">
          <img
            v-if="item.img_url"
            :src="item.img_url"
            alt
            @tap="prewImg"
            @longpress="longpress"
            :data-img="item.img_url"
            :data-index="index"
          >
          <img
            v-else
            src="../../../static/images/picture.svg"
            alt
            @tap="prewImg"
            @longpress="longpress"
            :data-img="item.img_url"
            :data-index="index"
          >
          <div
            @tap="addDesc"
            :data-index="index"
            :data-desc="item.desc"
            data-change="true"
          >{{item.desc || '点击输入文字'}}</div>
        </div>
        <div class="imgs-wrapper">
          <img
            src="../../../static/images/upload_img.svg"
            alt
            @tap="uploadImg"
            :data-index="index+1"
          >
          <img src="../../../static/images/text.svg" alt @tap="addDesc" :data-index="index+1">
        </div>
      </div>
    </div>

    <button @tap="submit" class="submit">确定发布</button>
    <modal :showModal="show" :close="close">
      <div v-for="(item, index) in categroys" :key="index" class="categroy-item">
        <div>{{item.value}}</div>
        <div>
          <span
            @tap.stop="select"
            v-for="(ele, i) in item.children"
            :key="i"
            :class="active.out_index == index && active.inner_index == i? 'active-item':''"
            :data-out-index="index"
            :data-inner-index="i"
            :data-ele="ele"
          >{{ele.value}}</span>
        </div>
      </div>
    </modal>
  </div>
</template>

<script>
import modal from "../../components/modal";
import Bus from "../../utils/bus.js";
import { setTimeout } from "timers";
export default {
  data() {
    return {
      show: false,
      categroys: [],
      selected_cate: "",
      cate_id: null,
      active: {
        out_index: -1,
        inner_index: -1
      },
      title: "",
      sections: []
    };
  },
  components: {
    modal
  },
  methods: {
    toggleModal() {
      this.show = !this.show;
    },
    select(e) {
      const {
        outIndex,
        innerIndex,
        ele: { value, cate_id },
        ele
      } = e.currentTarget.dataset;
      this.active.out_index = outIndex;
      this.active.inner_index = innerIndex;
      this.selected_cate = value;
      this.cate_id = cate_id;
      this.close();
    },
    close() {
      this.show = false;
    },
    getCategroy() {
      const db = wx.cloud.database(),
        categroys = db.collection("categroy");
      categroys.get().then(res => {
        this.categroys = res.data;
      });
    },
    bindKeyInput(e) {
      const { value } = e.mp.detail;
      this.title = value;
    },
    longpress(e) {
      this.uploadImg(e, "change");
    },
    uploadImg(e, type) {
      const { index } = e.currentTarget.dataset;
      wx.chooseImage({
        count: 1,
        sizeType: ["compressed"],
        sourceType: ["album", "camera"],
        success: function(res) {
          // tempFilePath可以作为img标签的src属性显示图片
          const tempFilePath = res.tempFilePaths[0];
          this.saveImg(tempFilePath, fileId =>
            this.changeView(tempFilePath, index, type, fileId)
          );
        }.bind(this)
      });
    },
    changeView(tempFilePath, index, type, fileId) {
      if (type == "change") {
        let desc = this.sections[index].desc;
        this.sections.splice(index, 1, {
          img_url: tempFilePath,
          desc,
          fileId
        });
      } else {
        this.sections.splice(index, 0, {
          img_url: tempFilePath,
          desc: "",
          fileId
        });
      }
    },
    saveImg(path, cb) {
      let reg = /\.\w+/g,
        matches = path.match(reg),
        file_type = matches[matches.length - 1];
      wx.showLoading({
        title: "图片正在上传中，请稍候"
      });
      wx.cloud
        .uploadFile({
          cloudPath: `articles/pictures/${Date.now()}-${Math.floor(
            Math.random(0, 1) * 10000000
          )}${file_type}`,
          filePath: path // 文件路径
        })
        .then(res => {
          wx.hideLoading();
          cb && cb(res.fileID);
        })
        .catch(error => {
          console.log(error);
        });
    },
    prewImg(e) {
      const { img } = e.currentTarget.dataset;
      if (!img) {
        this.uploadImg(e, "change");
        return;
      }
      wx.previewImage({
        current: img, // 当前显示图片的http链接
        urls: [img] // 需要预览的图片http链接列表
      });
    },
    addDesc(e) {
      const { index, desc = "", change = "" } = e.currentTarget.dataset;
      wx.navigateTo({
        url: `/pages/text/main?index=${index}&desc=${desc}&change=${change}`
      });
    },
    validate(terms) {
      return terms.every(item => {
        if (!item.rule) {
          wx.showToast({
            title: item.tip,
            icon: "none",
            duration: 1500
          });
        }
        return item.rule;
      });
    },
    submit() {
      if (
        !this.validate([
          {
            rule: this.selected_cate,
            tip: "请选择分类"
          },
          {
            rule: this.title,
            tip: "请输入标题"
          },
          {
            rule: this.sections.length,
            tip: "文章内容不能为空"
          }
        ])
      )
        return;
      const db = wx.cloud.database();
      db
        .collection("articles")
        .add({
          data: {
            cate_id: this.cate_id,
            title: this.title,
            section: this.sections.map(item => ({
              fileID: item.fileId,
              desc: item.desc
            }))
          }
        })
        .then(res => {
          wx.showToast({
            title: "发布成功",
            icon: "success",
            duration: 1500
          });
          setTimeout(() => {
            wx.switchTab({
              url: "/pages/index/main"
            });
          }, 1500);
        })
        .catch(err => {
          console.log(err);
        });
    }
  },
  created() {
    this.getCategroy();
    Bus.$on("new", info => {
      // 重新进入 清空数据
      if (info.type === "new") {
        this.sections = [];
        this.selected_cate = "";
        this.cate_id = null;
        this.active = {
          out_index: -1,
          inner_index: -1
        };
        this.title = "";
        return;
      }
    });
    Bus.$on("setContent", target => {
      const { index, content, change } = target;
      if (change) {
        let { img_url: url, fileId } = this.sections[index];
        this.sections.splice(index, 1, {
          img_url: url,
          desc: content,
          fileId
        });
      } else {
        this.sections.splice(index, 0, {
          img_url: "",
          desc: content
        });
      }
    });
  },
  onHide() {
    this.close();
  },
  onUnload() {
    this.close();
  }
};
</script>

<style lang="scss" scoped>
.wrapper {
  padding: 32rpx;
  .imgs-wrapper {
    display: flex;
    justify-content: center;
    img {
      width: 64rpx;
      height: 64rpx;
      &:first-of-type {
        margin-right: 20px;
        width: 80rpx;
        height: 80rpx;
      }
    }
  }
  .section-list {
    padding-bottom: 60px;
  }
  .section {
    display: flex;
    border: 1px solid #ccc;
    border-radius: 4px;
    padding: 10px;
    font-size: 14px;
    > div {
      flex: 1;
      margin-left: 10px;
    }
    img {
      width: 128rpx;
      height: 128rpx;
    }
  }
  > .categroy,
  .title,
  .imgs-wrapper {
    font-size: 14px;
    padding: 24rpx 0;
    display: flex;
  }
  .flex-item {
    flex: 1;
    margin-left: 0.5em;
    box-shadow: 0px 1px 4px 2px rgba(0, 0, 0, 0.04);
  }
}
.categroy {
  span {
    display: inline-block;
    flex: 0.5;
    height: 80rpx;
    line-height: 80rpx;
    text-align: center;
    border: 1px solid #ededed;
    border-radius: 2px;
    font-size: 12px;
    box-sizing: border-box;
  }
}
.title {
  input {
    display: inline-block;
    padding-left: 5px;
    width: 100%;
    height: 80rpx;
    line-height: 80rpx;
    border: 1px solid #ededed;
    border-radius: 2px;
    box-sizing: border-box;
  }
}
.categroy-item {
  font-size: 14px;
  margin-bottom: 10px;
  > div {
    &:first-of-type {
      font-size: 16px;
      font-weight: 500;
    }
  }
  span {
    pointer-events: auto;
    margin: 15px 15px 15px 0;
    border: 1px solid #444;
    border-radius: 2px;
    padding: 2px 10px;
    color: #444;
    display: inline-block;
  }
  .active-item {
    color: #33c5b9;
    border: 1px solid #33c5b9;
  }
}
</style>

