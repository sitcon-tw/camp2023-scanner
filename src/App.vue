<template>
  <template v-if="mode === 'student'">
    <p style="text-align: center" v-if="isAndroid">
      如果使用 Android，請點擊上面三個點 開啟於...
    </p>
    <p style="
        text-align: center;
        border: solid #69a14f 1px;
        border-radius: 5px;
        padding: 10px 5px;
        background-color: #77b55a;
        color: white;
      " v-if="hasAlert">
      {{ alertMsg }}
    </p>
    <QrcodeStream :onDetect="OnSuccess" />
  </template>
  <template v-else-if="mode === 'admin'">
    <h1 class="title">點數 QRCode 生成</h1>
    <div class="form">
      <div class="reason">
        <span class="reason-title">原因</span>
        <select v-model="description" class="reason-selection">
          <option value="" selected disabled>請選擇</option>
          <option>認真參與活動</option>
          <option>勇敢探索攤位</option>
          <option>上課表現卓越</option>
          <option>主動協助活動</option>
          <option>完成每日任務</option>
          <option>完成營期任務</option>
          <option>無故鬧事</option>
          <option>自訂</option>
        </select>
        <input type="text" v-model="custom" class="reason-custom" v-if="description === '自訂'" />
      </div>

      <div class="money">
        <span class="money-title">增減</span>
        <select v-model="upDown" class="money-selection">
          <option>獲得</option>
          <option>扣除</option>
        </select>
      </div>

      <div class="money">
        <span class="money-title">金額</span>
        <select v-model="point" class="money-selection">
          <option selected disabled>0</option>
          <option>100</option>
          <option>200</option>
          <option>300</option>
          <option>400</option>
          <option>500</option>
          <option>600</option>
          <option>700</option>
          <option>800</option>
          <option>900</option>
          <option>1000</option>
          <option>1100</option>
          <option>1200</option>
          <option>1300</option>
          <option>1400</option>
          <option>1500</option>
          <option>1600</option>
          <option>1700</option>
          <option>1800</option>
          <option>1900</option>
          <option>2000</option>
        </select>
      </div>
    </div>
    <h3 class="content">
      {{
        description === "自訂"
          ? custom
          : description === ""
            ? "未知原因"
            : description
      }}，{{ upDown }} {{ point }} 拉麵靈魂
    </h3>

    <button type="submit" @click="generate" class="submit-btn">
      <p class="submit-content">產生 QRCode</p>
    </button>
  </template>
  <template v-else-if="mode === 'admin-finish'">
    <h1 class="title">點數 QRCode 生成</h1>
    <div v-for="(item, index) in coupon" :key="index">
      <img :src="item" alt="QR Code" />
    </div>
    <div class="back-div">
      <button @click="back" class="back-btn">
        <p class="back-content">返回</p>
      </button>
    </div>
  </template>
  <template v-else-if="mode === 'god'">
    <div style="margin: 0 auto">
      <div style="display: inline-block; margin-right: 24px">
        <p>👀 解題狀況</p>
        <table>
          <tr v-for="(item, index) in problems" :key="'pro' + index">
            <td>{{ "第 " + (index + 1) + " 題" }}</td>
            <td></td>
            <td align="left">
              <code>{{ item.keyword ? item.keyword : "" }}</code>
            </td>
            <td>{{ item.solved_team.length }} 組</td>
          </tr>
        </table>
      </div>
      <div style="display: inline-block; margin-left: 24px">
        <p>🔍 各組解題狀況</p>
        <table style="margin: 0 auto">
          <tr v-for="(value, key) in groupProblem" :key="key">
            <td>{{ key }}</td>
            <td width="20" align="right"></td>
            <td>{{ value }} 題</td>
          </tr>
        </table>
      </div>
    </div>
  </template>
  <template v-else-if="mode === 'score-finish'">
    <div>
      <div v-if="status.length > 0">
        <table style="
            margin: 0 auto;
            text-align: center;
            padding: 15px 30px;
            border-radius: 15px;
            background-color: white;
            color: black;
          ">
          <thead>
            <th style="padding: 0 10px;">組別</th>
            <th style="padding: 0 15px;">分數</th>
            <th style="padding: 0 3px;">排名</th>
          </thead>
          <tr v-for="item in group" :key="item.name" class="table-row">
            <td>{{ item.name }}</td>
            <td>{{ rankStatus.find(s => s.name === item.name)?.coin || 0 }}</td>
            <td>
              <span v-if="getTeanRank(rankStatus.find(s => s.name === item.name)?.coin || 0) === 1"
                style="font-size: 16px; user-select: none; cursor: pointer;" @click="callFireworks('第一名')">🥇</span>
              <span v-else-if="getTeanRank(rankStatus.find(s => s.name === item.name)?.coin || 0) === 2"
                style="font-size: 16px; user-select: none; cursor: pointer;" @click="callFireworks('第二名')">🥈</span>
              <span v-else-if="getTeanRank(rankStatus.find(s => s.name === item.name)?.coin || 0) === 3"
                style="font-size: 16px; user-select: none; cursor: pointer;" @click="callFireworks('第三名')">🥉</span>
            </td>
          </tr>
        </table>
        <FireworksModal ref="fireworksModal" />
        <CoinTrendChart :staff_token="staff_token" />
      </div>
      <div v-else style="
          text-align: center;
          border: solid #69a14f 1px;
          border-radius: 5px;
          padding: 15px 30px;
          background-color: white;
          color: black;
          width: 10rem;
          margin: 0 auto;
        ">
        ⚠️ No permission.
      </div>
    </div>
  </template>
  <template v-else-if="mode === 'multi'">
    <h1 class="title">點數 QRCode 生成</h1>
    <div class="form">
      <div class="reason">
        <span class="reason-title">原因</span>
        <select v-model="description" class="reason-selection">
          <option value="" selected disabled>請選擇</option>
          <option>認真參與活動</option>
          <option>勇敢探索攤位</option>
          <option>上課表現卓越</option>
          <option>主動協助活動</option>
          <option>完成每日任務</option>
          <option>完成營期任務</option>
          <option>無故鬧事</option>
          <option>自訂</option>
        </select>
        <input type="text" v-model="custom" class="reason-custom" v-if="description === '自訂'" />
      </div>

      <div class="money">
        <span class="money-title">增減</span>
        <select v-model="upDown" class="money-selection">
          <option>獲得</option>
          <option>扣除</option>
        </select>
      </div>

      <div class="money">
        <span class="money-title">金額</span>
        <select v-model="point" class="money-selection">
          <option selected disabled>0</option>
          <option>100</option>
          <option>200</option>
          <option>300</option>
          <option>400</option>
          <option>500</option>
          <option>600</option>
          <option>700</option>
          <option>800</option>
          <option>900</option>
          <option>1000</option>
          <option>1100</option>
          <option>1200</option>
          <option>1300</option>
          <option>1400</option>
          <option>1500</option>
          <option>1600</option>
          <option>1700</option>
          <option>1800</option>
          <option>1900</option>
          <option>2000</option>
        </select>
      </div>
      <div class="count">
        <span class="count-title">數量</span>
        <select v-model="count" class="count-selection">
          <option selected>1</option>
          <option>2</option>
          <option>3</option>
          <option>4</option>
          <option>5</option>
          <option>6</option>
          <option>7</option>
          <option>8</option>
          <option>9</option>
          <option>10</option>
          <option>11</option>
          <option>12</option>
          <option>13</option>
          <option>14</option>
          <option>15</option>
          <option>16</option>
          <option>17</option>
          <option>18</option>
          <option>19</option>
          <option>20</option>
        </select>
      </div>
    </div>

    <h3 class="content">
      {{
        description === "自訂"
          ? custom
          : description === ""
            ? "未知原因"
            : description
      }}，{{ upDown }} {{ point }} 拉麵靈魂 => {{ count }} 份
    </h3>

    <button type="submit" @click="generate" class="submit-btn">
      <p class="submit-content">產生 QRCode</p>
    </button>
  </template>
  <template v-else>
    <div v-if="staff_token == null" style="
        text-align: center;
        border: solid #69a14f 1px;
        border-radius: 5px;
        padding: 15px 30px;
        background-color: white;
        color: black;
        width: 5rem;
        margin: 0 auto;
        font-weight: bold;
        font-size: 1.2rem;
      ">
      Loading...
    </div>
  </template>
</template>

<script>
import { QrcodeStream, QrcodeDropZone, QrcodeCapture } from "vue-qrcode-reader";
import axios from "axios";
import qs from "qs";

import CoinTrendChart from "./components/CoinTrendChart.vue";
import FireworksModal from "./components/FireworkModal.vue";

export default {
  name: "app",
  components: {
    QrcodeStream,
    QrcodeDropZone,
    QrcodeCapture,
    CoinTrendChart,
    FireworksModal,
  },
  chartData() {
    return; /* mutable chart data */
  },
  chartOptions() {
    return; /* mutable chart options */
  },
  data() {
    return {
      mode: "student",
      isAndroid: false,
      api: null,
      point: 0,
      description: "",
      upDown: "獲得",
      custom: "",
      coupon: [],
      count: 1,
      status: [],
      lock: false,
      hasAlert: false,
      alertMsg: "",
      staff_token: null,
      group: [
        {
          groupId: -1001588989706,
          name: "第一小隊",
        },
        {
          groupId: -1001897923106,
          name: "第二小隊",
        },
        {
          groupId: -1001989709878,
          name: "第三小隊",
        },
        {
          groupId: -1001964001299,
          name: "第四小隊",
        },
        {
          groupId: -1001915605809,
          name: "第五小隊",
        },
        {
          groupId: -1001720858102,
          name: "第六小隊",
        },
        {
          groupId: -1001984208872,
          name: "第七小隊",
        },
        {
          groupId: -1001826501562,
          name: "第八小隊",
        },
        {
          groupId: -1001967156783,
          name: "第九小隊",
        },
      ],
      problems: [],
    }
  },

  beforeMount() {
    var config = {
      baseURL: "https://camp-api.sitcon.party/",
      timeout: 3000,
    };
    this.api = axios.create(config);

    if (
      (this.parameters().multi || "").length !== 0 &&
      (this.parameters().token || "").length !== 0
    ) {
      this.mode = "multi";
    } else if ((this.parameters().token || "").length !== 0) {
      this.mode = "admin";
    } else if ((this.parameters().id || "").length !== 0) {
      this.mode = "student";
      this.isAndroid = navigator.userAgent.match(/Android/i);
    } else if ((this.parameters().god || "").length !== 0) {
      this.mode = "god";
      this.intervalHandler = this.executeAndSetInterval(
        function () {
          this.getProblem();
        }.bind(this),
        5000
      );
    } else {
      this.mode = "dashboard";
      this.intervalHandler = this.executeAndSetInterval(
        function () {
          this.getStatus();
        }.bind(this),
        5000
      );
    }
  },
  beforeDestroy() {
    window.clearInterval(this.intervalHandler);
  },
  computed: {
    desc: function () {
      if (this.description === "") {
        return "未知原因" + "，" + this.upDown;
      } else if (this.description !== "自訂") {
        return this.description + "，" + this.upDown;
      } else {
        return this.custom + "，" + this.upDown;
      }
    },
    groupProblem: function () {
      var result = {};
      for (var i = 0; i < this.problems.length; i++) {
        for (var j = 0; j < this.problems[i]["solved_team"].length; j++) {
          if (
            result[this.id2GroupName(this.problems[i]["solved_team"][j])] ===
            undefined
          ) {
            result[this.id2GroupName(this.problems[i]["solved_team"][j])] = 1;
          } else {
            result[this.id2GroupName(this.problems[i]["solved_team"][j])]++;
          }
        }
      }
      return result;
    },
    rankStatus: function () {
      return this.group.map(g => {
        const statusItem = this.status.find(s => s.name === g.name);
        return {
          name: g.name,
          coin: statusItem ? statusItem.coin : 0
        };
      }).sort((a, b) => b.coin - a.coin);
    },

    fireworks() {
      return this.$refs.fireworks;
    },
  },

  methods: {
    explode(element) {
      const distanceFromTop = window.scrollY + element.target.getBoundingClientRect().top;
      const screenHeight = window.innerHeight;
      const stageHeight = screenHeight - distanceFromTop + 100;
      this.explosions.push({
        stageHeight,
        particleCount: 90,
        duration: 3000,
      });
    },
    removeExplosion(index) {
      this.explosions.splice(index, 1);
    },
    OnSuccess(result) {
      result = result[0].rawValue;
      if (result.startsWith("http")) window.location.href = result;
      else if (!this.lock) {
        var self = this;
        if ((this.parameters().id || "").length !== 0) {
          this.lock = true;
          this.api
            .post(
              "consume",
              qs.stringify({ group_id: this.parameters().id, coupon: result })
            )
            .then(function (res) {
              self.hasAlert = true;
              self.alertMsg =
                "✅ " +
                (res.data.status == "OK"
                  ? "執行成功，可以關閉掃描器惹。"
                  : res.data.status);
              self.lock = false;
            })
            .catch(function (error) {
              self.hasAlert = true;
              self.alertMsg = "⚠️ 發生錯誤。 ➡️ " + error.response.data.message;
              console.log(error.message);
              self.lock = false;
            });
        }
      }
    },
    parameters() {
      return window.location.search
        .split("?")
        .pop()
        .split("&")
        .map(function (p) {
          var ps = p.split("=");
          var o = {};
          o[ps.shift()] = ps.join("=");
          return o;
        })
        .reduce(function (a, b) {
          var o = a;
          for (var k in b) {
            o[k] = b[k];
          }
          return o;
        });
    },
    generate() {
      const qrcode = [];
      if ((this.parameters().token || "").length !== 0) {
        if (this.mode === "multi") {
          for (let i = 0; i < this.count; i++) {
            this.api
              .post(
                "generate",
                qs.stringify({
                  token: this.parameters().token,
                  coin: this.point * (this.upDown === "獲得" ? 1 : -1),
                  description: this.desc,
                })
              )
              .then((res) => {
                qrcode.push("https://quickchart.io/qr?text=" + res.data.coupon);
                // console.log(
                //   "QR Code URL: ",
                //   "https://quickchart.io/qr?text=" + res.data.coupon
                // );
                if (i === this.count - 1) {
                  this.coupon = qrcode;
                  // console.log("Coupon Array: ", this.coupon);
                  this.mode = "admin-finish";
                }
              })
              .catch((error) => {
                console.error("Error generating QR code:", error);
              });
          }
        } else {
          this.api
            .post(
              "generate",
              qs.stringify({
                token: this.parameters().token,
                coin: this.point * (this.upDown === "獲得" ? 1 : -1),
                description: this.desc,
              })
            )
            .then((res) => {
              qrcode.push("https://quickchart.io/qr?text=" + res.data.coupon);
              // console.log(
              //   "QR Code URL: ",
              //   "https://quickchart.io/qr?text=" + res.data.coupon
              // );
              this.coupon = qrcode;
              // console.log("Coupon Array: ", this.coupon);
              this.mode = "admin-finish";
            })
            .catch((error) => {
              console.error("Error generating QR code:", error);
            });
        }
      }
    },
    back() {
      if (this.parameters().multi) this.mode = "multi";
      else this.mode = "admin";
    },
    getStatus() {
      var self = this;
      // console.log(self.staff_token);
      self.staff_token = this.parameters().staff_token;
      // console.log(self.staff_token);

      this.api
        .get("status", {
          params: { staff_token: this.parameters().staff_token },
        })
        .then(function (res) {
          self.status = res.data;
        })
        .then((this.mode = "score-finish"));
    },
    getProblem() {
      var self = this;
      this.api
        .get("keyword_status", {
          params: { staff_token: this.parameters().staff_token },
        })
        .then(function (res) {
          self.problems = res.data;
        });
    },
    getTeanRank(coin) {
      return this.rankStatus.findIndex(item => item.coin == coin) + 1;
    },
    id2GroupName(id) {
      var result = this.group.filter(function (element) {
        return element.groupId === id;
      });
      // console.log(result[0].name)
      if (result.length !== 0) return result[0].name;
      else return "";
    },
    executeAndSetInterval(callback, regularInterval) {
      callback();
      return setInterval(function () {
        callback();
      }, regularInterval);
    },
    callFireworks(ranktext) {
      this.$refs.fireworksModal.startFireworks(ranktext);
    },
  },
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

.qrcode-stream-wrapper {
  width: min(90vw, 90vh);
  max-width: 512px;
  aspect-ratio: 1/1;
  overflow: hidden;
  box-shadow: 2.8px 2.8px 2.2px rgba(0, 0, 0, 0.02),
    6.7px 6.7px 5.3px rgba(0, 0, 0, 0.028),
    12.5px 12.5px 10px rgba(0, 0, 0, 0.035),
    22.3px 22.3px 17.9px rgba(0, 0, 0, 0.042),
    41.8px 41.8px 33.4px rgba(0, 0, 0, 0.05),
    100px 100px 80px rgba(0, 0, 0, 0.07);
  border-radius: 16px;
  position: relative;

  .qrcode-stream-camera {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
}
</style>
