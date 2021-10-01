<template>
  <div>
    <div class="header">
      <div v-if="isLogin">
        <div class="header-title">登录</div>
        <div class="header-info">Please login your account</div>
      </div>
      <div v-else>
        <div class="header-title">注册</div>
        <div class="header-info">Please register an account</div>
      </div>
    </div>
    <div class="body">
      <div class="login-form">
        <van-field
          placeholder="请输入用户名"
          :value="inputUserName"
          @change="onUserNameChange"
        />
        <van-field
          placeholder="请输入密码"
          :value="inputPassword"
          type="password"
          @change="onPasswordChange"
        />
        <div v-if="!isLogin">
          <van-field
            placeholder="找回密码手机号"
            :value="inputKeys"
            @change="onKeysChange"
          />
        </div>
      </div>
      <van-button slot="button" round block color="#4d80f0" @click="onClick">
        {{ isLogin ? "登录" : "注册" }}
      </van-button>
      <div class="other-option">
        <div @click="onOptionClick">
          <span>{{ isLogin ? "注册账号" : "登录账号" }}</span>
        </div>
        <span style="margin: 0 30px">|</span>
        <div @click="onForgetClick">
          <span>忘记密码</span>
        </div>
      </div>
      <div>
        <van-button
          style="display: flex; justify-content: center; margin-top: 20px"
          slot="button"
          round
          color="#a0a0a0"
          @click="onInterClick"
        >
          直接进入
        </van-button>
      </div>
      <div class="copyright-wrapper">
        <span class="copyright">Power by Ewing</span>
      </div>
      <van-dialog
        use-slot
        title="找回密码"
        :show="ForgetPswd"
        show-cancel-button
        transition="fade"
        @confirm="onFindPwConfirm"
        @cancel="onFindPwCancel"
      >
        <van-field
          label="手机号"
          title-weight="60px"
          placeholder="请输入找回密码手机号"
          :value="inputKeys"
          @change="onKeysChange"
        />
      </van-dialog>
      <van-toast id="van-toast" />
    </div>
  </div>
</template>

<script>
import Toast from "vant-weapp/dist/toast/toast";
export default {
  data() {
    return {
      isLogin: true,
      inputUserName: "",
      inputPassword: "",
      inputKeys: "",
      ForgetPswd: false,
    };
  },
  methods: {
    onUserNameChange(event) {
      var that = this;
      that.$set(that, "inputUserName", event.mp.detail);
      console.log("输入的账号:", event.mp.detail);
    },
    onPasswordChange(event) {
      var that = this;
      that.$set(that, "inputPassword", event.mp.detail);
      console.log("输入的密码:", event.mp.detail);
    },
    onKeysChange(event) {
      var that = this;
      that.$set(that, "inputKeys", event.mp.detail);
      console.log("输入的手机号:", event.mp.detail);
    },
    onClick(event) {
      var that = this;
      if (
        that.inputUserName != "" &&
        that.inputPassword != "" &&
        that.inputKeys != ""
      ) {
        wx.showToast({
          title: that.isLogin ? "登录中" : "注册中",
          icon: "loading",
          duration: 2000,
          mask: true,
        });
        //模拟请求服务器延时500ms
        setTimeout(() => {
          //箭头函数，JS的ES6语法
          if (!that.isLogin) {
            wx.setStorage({
              key: "user",
              data: {
                username: that.inputUserName,
                password: that.inputPassword,
                contact: that.inputKeys,
              },
              success(res) {
                wx.hideToast();
                wx.showModal({
                  title: "注册成功",
                  content: "是否立即登录？",
                  success(res) {
                    if (res.confirm) {
                      that.isLogin = !that.isLogin;
                      console.log("用户点击确定");
                    }
                  },
                });
              },
              fail(res) {
                Toast.success("注册失败");
              },
            });
          } else {
            wx.getStorage({
              key: "user",
              success(res) {
                if (
                  that.inputUserName === res.data.username &&
                  that.inputPassword === res.data.password
                ) {
                  wx.showToast({
                    title: "登录成功",
                    icon: "success",
                    duration: 2000,
                    mask: true,
                  });
                  if (that.inputUserName === "yuyone") {
                    setTimeout(() => {
                      wx.switchTab({
                        url: "/pages/index/main",
                      });
                    }, 500);
                  } else {
                    console.log("非本人");
                    wx.showToast({
                      title: "没有权限访问，需要得到管理员许可才可进入主页",
                      icon: "none",
                      duration: 2000,
                    });
                  }
                } else {
                  wx.showToast({
                    title: "登录失败，用户名或密码错误",
                    icon: "none",
                    duration: 3000,
                  });
                }
              },
              fail(res) {
                wx.showToast({
                  title: "登录失败",
                  icon: "error",
                  duration: 2000,
                });
              },
            });
          }
        }, 500);
      } else {
        wx.showToast({
          title: "输入不能为空！",
          icon: "none",
          duration: 2000,
        });
      }
    },
    onOptionClick(event) {
      var that = this;
      that.isLogin = !that.isLogin;
      wx.setNavigationBarTitle({
        title: that.isLogin ? "登录" : "注册",
      });
    },
    onForgetClick(event) {
      var that = this;
      that.ForgetPswd = true;
    },
    onFindPwConfirm(event) {
      var that = this;
      wx.getStorage({
        key: "user",
        success(res) {
          if (that.inputKeys === res.data.contact) {
            wx.showModal({
              title: "请牢记密码",
              content: res.data.password,
              showCancel: false,
              confirmText: "立即登录",
              success(res) {
                if (res.confirm) {
                  that.isLogin = true;
                }
              },
            });
          } else {
            Toast.fail("没有查到此手机号的注册信息");
          }
        },
        fail(res) {
          Toast.fail("没有找到您的信息");
        },
      });
      that.ForgetPswd = false;
    },
    onFindPwCancel(event) {
      var that = this;
      that.ForgetPswd = false;
    },
    onInterClick(event) {
      wx.switchTab({
        url: "/pages/index/main",
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
    color: #fff;
  }
  .header-info {
    font-size: 12px;
    margin-left: 20px;
    color: #eaeaea;
  }
}
.body {
  padding: 20px;
  margin-top: -20px;
  border-radius: 20px 20px 0 0;
  background-color: #fff;
  .login-form {
    margin-bottom: 30px;
  }
  .other-option {
    display: flex;
    justify-content: center;
    margin-top: 20px;
    color: #9f9f9f;
  }
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
