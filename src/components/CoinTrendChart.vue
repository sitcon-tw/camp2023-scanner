<template>
  <div class="main">
    <div class="chart-container">
      <canvas ref="coinTrendChart"></canvas>
    </div>
  </div>
  <div class="button-container">
    <button class="reset-chart-button" @click="resetZoom">重製圖表</button>
  </div>
</template>

<script>
import { Chart, registerables } from "chart.js";
import "chartjs-adapter-date-fns";
import 'chartjs-adapter-moment';
import ChartDataLabels from "chartjs-plugin-datalabels";
import zoomPlugin from 'chartjs-plugin-zoom';

Chart.register(...registerables, ChartDataLabels, zoomPlugin);

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
        "#CCCC00", // 黃色
        "#008000", // 綠色
        "#0000FF", // 藍色
        "#4B0082", // 靛色
        "#EE82EE", // 紫色
        "#000000", // 黑色
        "#808080", // 灰色
      ],
      teamOrder: [
        "第一小隊", // 紅色
        "第二小隊", // 橙色
        "第三小隊", // 黃色
        "第四小隊", // 綠色
        "第五小隊", // 藍色
        "第六小隊", // 靛色
        "第七小隊", // 紫色
        "第八小隊", // 黑色
        "第九小隊", // 灰色
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
          // 分組並累積點數
          const groupedData = this.processData(data);
          this.renderChart(groupedData);
        })
        .catch(error => console.error("Error:", error));
    },    

    processData(data) { 
       // 按時間戳排序
       data.sort((a, b) => a.consume_timestamp - b.consume_timestamp);
      
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

          return groupedData;
    },
          
    renderChart(groupedData) {
      const ctx = this.$refs.coinTrendChart.getContext("2d");
      const datasets = this.teamOrder
        .filter((teamName) => groupedData[teamName]) // 確保小隊存在於數據中
        .map((teamName, index) => ({
          label: teamName,
          data: groupedData[teamName],
          fill: false,
          stepped: true,
          borderColor: this.teamColors[index],
          backgroundColor: this.teamColors[index],
          pointRadius: (context) => {
            const point = context.raw;
            return point.reason !== "" ? 3 : 0;
          },
          pointHoverRadius: (context) => {
            const point = context.raw;
            return point.reason !== "" ? 6 : 0;
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
              enabled: true,
              callbacks: {
                label: function (context) {
                  const point = context.raw;
                  // Check if 'reason' is not empty, then only return the label.
                  if (point.reason !== "") {
                    return `${context.dataset.label}: ${point.y}點 - ${point.reason} (${point.points}點)`;
                  } else {
                    return null; // Return null to prevent the tooltip from displaying
                  }
                },
              },
            },
            datalabels: {
              display: false, // 不顯示數據標籤
            },
            zoom: {
              zoom: {
                wheel: { enabled: true },
                pinch: { enabled: true },
                mode: 'x',
                drag: {
                  enabled: true,
                  backgroundColor: 'rgba(225,225,225,0.5)',
                  borderColor: 'rgba(225,225,225)',
                  borderWidth: 1,
                  threshold: 10
                }
              },
              pan: {
                enabled: true,
                mode: 'x'
              },
              limits: {
                x: {min: 'original', max: 'original'}
              }
            }
          },
        },
      });
    },
    resetZoom() {
      if (this.chart) {
        this.chart.resetZoom();
      }
    },
    beforeUnmount() {
      if (this.chart) {
        this.chart.destroy();
      }
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

.button-container {
  display: flex;
  justify-content: center;
  margin-top: 1rem;
}

.reset-chart-button {
  border-radius: 2rem;
  background-color: black;
  color: white;
  font-size: 1rem;
  padding: 0.5rem 1rem;
  border: 2px solid white;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
}

.reset-chart-button:active {
  transform: translateY(0);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

@media (max-width: 768px) {
  .chart-container {
    height: 300px; /* 調整手機上的高度 */
  }

  .reset-chart-button {
    font-size: 0.9rem;
    padding: 0.4rem 0.8rem;
  }
}

@media (max-width: 480px) {
  .chart-container {
    height: 250px; /* 調整小屏幕上的高度 */
  }

  .reset-chart-button {
    font-size: 0.8rem;
    padding: 0.3rem 0.6rem;
  }
}
</style>
