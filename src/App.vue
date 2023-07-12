<template>
  <div id="app">
    <template v-if="mode === 'student'">
      <p style="text-align: center" v-if="isAndroid">
        如果使用 Android，請點擊上面三個點 開啟於...
      </p>
      <QrcodeStream @onDecode="OnSuccess" style="width: 200px; height: 200px" />
      <QrcodeDropZone />
      <QrcodeCapture />
    </template>
    <template v-else-if="mode === 'admin'">
      <p style="text-align: center">{{ point >= 0 ? "獲得 " : "" }}分數：</p>
      <input
        type="number"
        max="1000"
        min="-100000000"
        step="100"
        list="defaultNumbers"
        style="
          text-align: center;
          width: 150px;
          height: 150px;
          font-size: 1.6rem;
        "
        v-model="point"
      />
      <datalist id="defaultNumbers">
        <option value="1000"></option>
        <option value="500"></option>
        <option value="200"></option>
        <option value="100"></option>
        <option value="-100"></option>
        <option value="-200"></option>
        <option value="-500"></option>
        <option value="-1000"></option>
      </datalist>
      <p>
        <select v-model="description">
          <option>認真參與活動，{{ point >= 0 ? "獲得 " : "" }}</option>
          <option>勇敢探索攤位，{{ point >= 0 ? "獲得 " : "" }}</option>
          <option>上課表現卓越，{{ point >= 0 ? "獲得 " : "" }}</option>
          <option>無故鬧事，{{ point >= 0 ? "獲得 " : "" }}</option>
          <option>自訂</option>
        </select>
      </p>
      <p v-if="description == '自訂'">
        <input type="text" v-model="custom" />{{ point >= 0 ? "獲得 " : "" }}
      </p>
      <button type="submit" @click="generate">產生</button>
    </template>
    <template v-else-if="mode === 'admin-finish'">
      <img :src="coupon" />
      <p>
        <button @click="back">返回</button>
      </p>
    </template>
    <template v-else-if="mode === 'god'">
      <div style="margin: 0 auto">
        <div style="display: inline-block; margin-right: 24px">
          <p>解題狀況</p>
          <table>
            <tr v-for="(item, index) in problems" :key="'pro' + index">
              <td>{{ "第" + (index + 1) + "題" }}</td>
              <td></td>
              <td>{{ item.solved_team.length }}</td>
            </tr>
          </table>
        </div>
        <div style="display: inline-block; margin-left: 24px">
          <p>各組解題狀況</p>
          <table style="margin: 0 auto">
            <tr v-for="(value, key) in groupProblem" :key="key">
              <td>{{ key }}</td>
              <td></td>
              <td>{{ value }}</td>
            </tr>
          </table>
        </div>
      </div>
    </template>
    <template v-else>
      <table style="margin: 0 auto">
        <tr v-for="item in status" :key="item['group_id']">
          <td>{{ item.name }}</td>
          <td>{{ item.coin }}</td>
        </tr>
      </table>
    </template>
  </div>
</template>

<script>
import {
  QrcodeStream,
  QrcodeDropZone,
  QrcodeCapture,
} from "qrcode-reader-vue3";
import axios from "axios";
import qs from "qs";

export default {
  name: "app",
  components: {
    QrcodeStream,
    QrcodeDropZone,
    QrcodeCapture,
  },
  data() {
    return {
      mode: "student",
      isAndroid: false,
      api: null,
      point: 0,
      description: "",
      custom: "",
      coupon: "",
      status: [],
      lock: false,
      group: [
        {
          groupId: -1001221619597,
          name: "第一小隊",
        },
        {
          groupId: -1001376339069,
          name: "第二小隊",
        },
        {
          groupId: -1001461040548,
          name: "第三小隊",
        },
        {
          groupId: -1001457570830,
          name: "第四小隊",
        },
        {
          groupId: -1001345724637,
          name: "第五小隊",
        },
        {
          groupId: -1001402246405,
          name: "第六小隊",
        },
        {
          groupId: -1001380073416,
          name: "第七小隊",
        },
        {
          groupId: -1001342649480,
          name: "第八小隊",
        },
        {
          groupId: -1001387397358,
          name: "第九小隊",
        },
        {
          groupId: -1001396420235,
          name: "第十小隊",
        },
      ],
      problems: [],
    };
  },
  beforeMount() {
    var config = {
      baseURL: "https://camp-api.sitcon.party/",
      timeout: 3000,
    };
    this.api = axios.create(config);

    if ((this.parameters().token || "").length !== 0) {
      this.mode = "admin";
    } else if ((this.parameters().id || "").length !== 0) {
      this.mode = "student";
      this.isAndroid = navigator.userAgent.match(/Android/i);
    } else if ((this.parameters().god || "").length !== 0) {
      this.mode = "god";
      window.setInterval(
        function () {
          this.getProblem();
        }.bind(this),
        1000
      );
    } else {
      this.mode = "dashboard";
      window.setInterval(
        function () {
          this.getStatus();
        }.bind(this),
        1000
      );
    }
  },
  computed: {
    desc: function () {
      if (this.description !== "自訂") {
        return this.description;
      } else {
        return this.custom + (this.point >= 0 ? "獲得 " : " ");
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
  },
  methods: {
    OnSuccess(result) {
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
              alert(res.data.status);
              self.lock = false;
            })
            .catch(function (error) {
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
      if ((this.parameters().token || "").length !== 0) {
        let data = JSON.stringify({
          token: this.parameters().token,
          coin: this.point,
          description: this.desc,
        });
        this.coupon =
          "https://chart.googleapis.com/chart?cht=qr&chl=" +
          data +
          "&chs=300x300";
        this.mode = "admin-finish";
      }
    },
    back() {
      this.mode = "admin";
    },
    getStatus() {
      var self = this;
      this.api.get("status").then(function (res) {
        self.status = res.data;
      });
    },
    getProblem() {
      var self = this;
      this.api.get("keyword_status").then(function (res) {
        self.problems = res.data;
      });
    },
    id2GroupName(id) {
      var result = this.group.filter(function (element) {
        return element.groupId === id;
      });
      // console.log(result[0].name)
      if (result.length !== 0) return result[0].name;
      else return "";
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
</style>
