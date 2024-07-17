<template>
  <div class="main">
    <canvas ref="coinTrendChart"></canvas>
  </div>
</template>

<script>
import { Chart, registerables } from "chart.js";
import "chartjs-adapter-date-fns";

Chart.register(...registerables);

export default {
  name: "CoinTrendChart",
  data() {
    return {
      chart: null,
      teamColors: [
        "#FF0000", // 紅色
        "#FFA500", // 橙色
        "#FFFF00", // 黃色
        "#008000", // 綠色
        "#0000FF", // 藍色
        "#4B0082", // 靛色
        "#EE82EE", // 紫色
        "#000000", // 黑色
        "#808080", // 灰色
      ],
    };
  },
  mounted() {
    this.fetchData();
  },
  methods: {
    fetchData() {
      fetch(
        "https://camp-api.sitcon.party/consume_history?staff_token=bef65917ecd1f48794c5ab32521df52be2b0ed39"
      )
        .then((response) => response.json())
        .then((data) => {
          // 檢查數據是否正確
          if (!Array.isArray(data)) {
            throw new Error("API 返回的數據格式不正確");
          }

          // 按時間戳排序
          data.sort((a, b) => a.consume_timestamp - b.consume_timestamp);

          // 分組並累積點數
          const groupedData = {};
          data.forEach((item) => {
            if (!groupedData[item.team_name]) {
              groupedData[item.team_name] = [];
            }
            const lastValue =
              groupedData[item.team_name].length > 0
                ? groupedData[item.team_name][
                    groupedData[item.team_name].length - 1
                  ].y
                : 0;
            groupedData[item.team_name].push({
              x: new Date(item.consume_timestamp * 1000),
              y: lastValue + item.coin,
            });
          });

          this.renderChart(groupedData);
        })
        .catch((error) => console.error("Error:", error));
    },
    renderChart(groupedData) {
      const ctx = this.$refs.coinTrendChart.getContext("2d");
      const datasets = Object.keys(groupedData).map((teamName, index) => ({
        label: teamName,
        data: groupedData[teamName],
        fill: false,
        stepped: true,
        borderColor: this.teamColors[index % this.teamColors.length],
        backgroundColor: this.teamColors[index % this.teamColors.length],
      }));

      this.chart = new Chart(ctx, {
        type: "line",
        data: {
          datasets,
        },
        options: {
          responsive: true,
          scales: {
            x: {
              type: "time",
              time: {
                unit: "minute",
              },
              title: {
                display: true,
                text: "時間",
              },
            },
            y: {
              title: {
                display: true,
                text: "累積點數",
              },
              beginAtZero: true,
            },
          },
          plugins: {
            title: {
              display: true,
              text: "小隊點數累積趨勢圖",
            },
          },
        },
      });
    },
  },
  beforeDestroy() {
    if (this.chart) {
      this.chart.destroy();
    }
  },
};
</script>

<style scoped>
canvas {
  max-width: 100%;
  height: 400px; /* 確保有高度 */
}

.main {
  margin-top: 2rem;
  background-color: white;
  border-radius: 1.5rem;
  padding: 0.75rem 0.75rem 0.75rem 0.75rem;
}
</style>
