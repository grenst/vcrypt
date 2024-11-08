<template>
    <div>
      <canvas ref="chart"></canvas>
    </div>
  </template>
  
  <script>
  import { Line } from 'vue-chartjs';
  import { Chart, registerables } from 'chart.js';
  Chart.register(...registerables);
  
  export default {
    extends: Line,
    props: {
      priceData: {
        type: Array,
        required: true
      }
    },
    data() {
      return {
        chartData: {
          labels: Array(this.priceData.length).fill(""),
          datasets: [
            {
              label: 'Price',
              data: this.priceData,
              borderColor: 'rgba(75, 192, 192, 1)',
              backgroundColor: 'rgba(75, 192, 192, 0.2)',
              fill: true,
              tension: 0.4
            }
          ]
        },
        chartOptions: {
          responsive: true,
          maintainAspectRatio: false,
          scales: {
            x: {
              display: false
            },
            y: {
              beginAtZero: false,
              ticks: {
                callback: function(value) {
                  return value;
                }
              }
            }
          },
          plugins: {
            legend: {
              display: false
            }
          }
        }
      };
    },
    watch: {
      priceData(newData) {
        this.chartData.datasets[0].data = newData;
        this.chartData.labels = Array(newData.length).fill("");
        this.$data._chart.update();
      }
    },
    mounted() {
      this.renderChart(this.chartData, this.chartOptions);
    }
  };
  </script>
  
  <style scoped>
  canvas {
    height: 100px;
  }
  </style>
  