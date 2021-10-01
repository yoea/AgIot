<template>
  <div class="wrapper">
    <div class="header-wrapper">
      <div class="header-title">
        <span>{{ city }}{{ area }} {{ feelsLike }}℃</span>
        <span>湿度 {{ humidity }}%</span>
      </div>
      <div class="header-text">
        <span>{{ temperature }}℃</span>
        <span>{{ weatherText }}</span>
      </div>
      <div class="weather-advice">
        风向：{{ wind_direction }}，风速：{{ wind_speed }}km/h，气压：{{
          pressure
        }}hPa
      </div>
    </div>
    <div class="body-wrapper">
      <div class="body">
        <div class="data-wrapper">
          <div class="data">
            <img class="data-logo" src="/static/images/temp.png" />
            <div class="data-text">
              <div class="data-title">温度</div>
              <div class="data-value">{{ Temp }}℃</div>
            </div>
          </div>
          <div class="data">
            <img class="data-logo" src="/static/images/hum.png" />
            <div class="data-text">
              <div class="data-title">湿度</div>
              <div class="data-value">{{ Hum }}%</div>
            </div>
          </div>
        </div>
        <div class="data-wrapper">
          <div class="data">
            <img class="data-logo" src="/static/images/ac.png" />
            <div class="data-text">
              <div class="data-title">空调</div>
              <div class="data-value">
                <switch @change="onACChange" :checked="AC" color="#3d7ef9" />
              </div>
            </div>
          </div>
          <div class="data">
            <img class="data-logo" src="/static/images/light.png" />
            <div class="data-text">
              <div class="data-title">灯光</div>
              <div class="data-value">
                <switch @change="onLedChange" :checked="Led" color="#3d7ef9" />
              </div>
            </div>
          </div>
        </div>
        <div class="copyright-wrapper">
          <span class="copyright">Power by Ewing</span>
        </div>
        <div style="margin-top: 15px" v-if="!isGotDevData">
          <van-notice-bar
            mode="closeable"
            text="正在获取设备采集的数据，请稍候..."
          />
        </div>
        <div style="display: flex; justify-content: center; margin-top: 20px">
          <van-button slot="button" round block color="#4d80f0" @click="onClick"
            >神奇按钮
          </van-button>
        </div>
        <div>
          <text>{{ ewingspage }}</text>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { connect } from "mqtt/dist/mqtt.js";

const mqttUrl = "wxs://pearl.ewing.top:8084/mqtt";
const options = {
  clientId: "emqx_yyy_client_e1",
};

export default {
  data() {
    return {
      client: {},
      ewingspage: "",
      isGotDevData: false,
      MqttStandone: false,
      Temp: 0,
      Hum: 0,
      Led: false,
      AC: false,
      city: "定位中", //城市
      area: "...", //区域
      humidity: 0,
      feelsLike: 0, //体感温度
      temperature: 0,
      wind_direction: "获取中",
      wind_speed: 0,
      weatherText: "获取中",
      pressure: 0,
    };
  },

  components: {},

  methods: {
    onACChange(event) {
      var that = this;
      console.log(event.mp.detail);
      let sw = event.mp.detail.value;
      if (sw) {
        that.client.publish("/esp/sub", '{"AC_SW":1}', function (err) {
          if (!err) {
            console.log("成功发命令：打开空调");
          }
        });
      } else {
        that.client.publish("/esp/sub", '{"AC_SW":0}', function (err) {
          if (!err) {
            console.log("成功发命令：关闭空调");
          }
        });
      }
    },
    onLedChange(event) {
      var that = this;
      console.log(event.mp.detail);
      let sw = event.mp.detail.value;
      if (sw) {
        that.client.publish("/esp/sub", '{"LED_SW":1}', function (err) {
          if (!err) {
            console.log("成功发命令：开灯");
          }
        });
      } else {
        that.client.publish("/esp/sub", '{"LED_SW":0}', function (err) {
          if (!err) {
            console.log("成功发命令：关灯");
          }
        });
      }
    },
    onClick(event) {
      var that = this;
      console.log("神奇按钮被按下");
      wx.request({
        url: "https://www.ewing.top/kaelvean/index.html",
        method: "get",
        success: function (res) {
          console.log(res.data);
          that.ewingspage = res.data;
        },
      });
      wx.showToast({
        title: "神奇按钮被按下",
        icon: "none",
        duration: 500,
      });
    },
  },
  onShow() {
    var that = this;
    wx.hideTabBar();
    if (that.MqttStandone != true) {
      that.client = connect(mqttUrl, options);
      that.client.on("connect", function () {
        console.log("成功连接mqtt服务器");
        that.client.subscribe("/esp/pub", function (err) {
          if (!err) {
            console.log("订阅Topic:/esp/pub成功");
            that.MqttStandone = true;
          } else {
            console.log("订阅/esp/pub失败");
          }
        });
        that.client.subscribe("/esp/set", function (err) {
          if (!err) {
            console.log("订阅Topic:/esp/set成功");
            that.MqttStandone = true;
          } else {
            console.log("订阅/esp/set失败");
          }
        });
      });
    }
    that.client.on("message", function (topic, message) {
      console.log(`收到来自：${topic}的消息：`);
      that.isGotDevData = true;
      //message是16进制buffer流
      let dataFromDev = {};
      dataFromDev = JSON.parse(message);
      //打印JSON格式message数据
      console.log(dataFromDev);
      //赋值给组件(显示)
      that.Temp = dataFromDev.Temp;
      that.Hum = dataFromDev.Hum;
      that.AC = dataFromDev.AC_SW;
      that.Led = dataFromDev.LED_SW;
    });
    wx.getLocation({
      type: "wgs84",
      success(res) {
        const latitude = res.latitude;
        const longitude = res.longitude;
        const key = "80434000c64b4c03a5aa68f7667711f3";
        wx.request({
          url: `https://devapi.qweather.com/s6/weather/now?location=${longitude},${latitude}&key=${key}`, //和风天气api
          success(res) {
            console.log(res.data);
            const { basic, now } = res.data.HeWeather6[0];
            console.log(basic);
            console.log(now);
            that.area = basic.location;
            that.city = basic.parent_city;
            that.feelsLike = now.fl;
            that.temperature = now.tmp;
            that.humidity = now.hum;
            that.weatherText = now.cond_txt;
            that.wind_direction = now.wind_dir;
            that.wind_speed = now.wind_spd;
            that.pressure = now.pres;
          },
        });
      },
    });
  },
};
</script>

<style lang="scss" scoped>
.wrapper {
  padding: 15px;
  .header-wrapper {
    background-color: #4d80f0;
    border-radius: 20px;
    color: #fff;
    box-shadow: #d6d6d6 0px 0px 5px;
    padding: 15px 30px;
    .header-title {
      display: flex;
      justify-content: space-between;
    }
    .header-text {
      font-size: 38px;
      font-weight: 400;
      display: flex;
      justify-content: space-between;
    }
    .weather-advice {
      margin-top: 10px;
      font-size: 12px;
    }
  }
  .data-wrapper {
    margin-top: 20px;
    display: flex;
    justify-content: space-between;
    .data {
      background-color: #fff;
      width: 150px;
      height: 80px;
      border-radius: 20px;
      display: flex;
      justify-content: space-around;
      padding: 0 8px;
      box-shadow: #d6d6d6 0px 0px 5px;
      .data-logo {
        height: 36px;
        width: 36px;
        margin-top: 22px;
      }
      .data-text {
        margin-top: 15px;
        color: #7f7f7f;
        .data-title {
          text-align: right;
        }
        .data-value {
          font-size: 26px;
        }
      }
    }
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
