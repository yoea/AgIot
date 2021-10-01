<template>
  <div>
    <div class="header">
      <div class="header-title">个人中心</div>
      <div class="header-info">User information</div>
    </div>
    <div class="body">
      <div class="copyright-wrapper">
        <span class="copyright">Power by Ewing</span>
      </div>
      <div>
        <van-button slot="button" round block color="#4d80f0" @click="onClick">神奇按钮
        </van-button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {};
  },
  methods: {
    onShow() {
      wx.setBackgroundColor({
        backgroundColorTop: "#4d80f0", // 顶部窗口的背景色
        backgroundColorBottom: "#ffffff", // 底部窗口的背景色
      });
    },
  },
};
</script>

<style lang="scss" scoped>
.header {
  height: 120px;
  width: 25px 0;
  background-color: #4d80f0;
  .header-title {
    font-size: 28px;
    font-weight: 500;
    margin-left: 20px;
    color: #ffffff;
  }
  .header-info {
    font-size: 12px;
    margin-left: 20px;
    color: #ffffff;
  }
}
.body {
  padding: 20px;
  margin-top: -20px;
  border-radius: 20px 20px 0 0;
  background-color: #fff;
  .copyright-wrapper {
    display: flex;
    justify-content: center;
    .copyright {
      font-size: 14px;
      color: #d5d5d5;
      position: fixed;
      bottom: 50px;
    }
  }
}
</style>
