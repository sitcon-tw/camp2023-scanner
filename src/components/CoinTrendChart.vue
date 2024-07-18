<template>
  <div class="main">
    <div class="chart-container">
      <canvas ref="coinTrendChart"></canvas>
    </div>
  </div>
</template>

<script>
import { Chart, registerables } from "chart.js";
import "chartjs-adapter-date-fns";
import ChartDataLabels from "chartjs-plugin-datalabels";

Chart.register(...registerables, ChartDataLabels);

export default {
  name: "CoinTrendChart",
  props: ["staff_token"],
  data() {
    return {
      staffToken: this.staff_token,
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
        `https://camp-api.sitcon.party/consume_history?staff_token=${this.staffToken}`
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
          const allTimestamps = new Set(
            data.map((item) => item.consume_timestamp)
          );

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
              reason: item.description,
              points: item.coin,
            });
          });

          // 為每個小隊填充空白時間戳
          Object.keys(groupedData).forEach((teamName) => {
            let lastValue = 0;
            const filledData = [];
            allTimestamps.forEach((timestamp) => {
              const existingPoint = groupedData[teamName].find(
                (point) => point.x.getTime() === timestamp * 1000
              );
              if (existingPoint) {
                lastValue = existingPoint.y;
                filledData.push(existingPoint);
              } else {
                filledData.push({
                  x: new Date(timestamp * 1000),
                  y: lastValue,
                  reason: "", // 空原因
                  points: 0, // 0 點數
                });
              }
            });
            groupedData[teamName] = filledData;
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
        pointRadius: (context) => {
          const point = context.raw;
          return point.points > 0 ? 3 : 0; // 有點數的地方顯示點，否則隱藏點
        },
        pointHoverRadius: (context) => {
          const point = context.raw;
          return point.points > 0 ? 6 : 0; // 有點數的地方顯示點，否則隱藏點
        },
      }));

      this.chart = new Chart(ctx, {
        type: "line",
        data: {
          datasets,
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
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
            tooltip: {
              callbacks: {
                label: function (context) {
                  const point = context.raw;
                  return `${context.dataset.label}: ${point.y}點 - ${point.reason} (${point.points}點)`;
                },
              },
            },
            datalabels: {
              display: false, // 不顯示數據標籤
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
.chart-container {
  position: relative;
  width: 100%;
  height: 400px; /* 確保有高度 */
}

.main {
  margin-top: 2rem;
  background-color: white;
  border-radius: 1.5rem;
  padding: 0.75rem 0.75rem 0.75rem 0.75rem;
}

@media (max-width: 768px) {
  .chart-container {
    height: 300px; /* 調整手機上的高度 */
  }
}

@media (max-width: 480px) {
  .chart-container {
    height: 250px; /* 調整小屏幕上的高度 */
  }
}
</style>
