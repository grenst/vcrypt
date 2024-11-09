<template>
    <div>
      <canvas ref="chart"></canvas>
    </div>
  </template>
  
  <script>
  import { Chart, registerables } from 'chart.js';
  Chart.register(...registerables);
  
  export default {
    props: {
      priceData: {
        type: Number,
        required: true
      }
    },
    data() {
      return {
        chart: null, // Экземпляр графика
        chartData: {
          labels: Array(20).fill(""), // Начальная настройка с пустыми метками
          datasets: [
            {
              label: 'Price',
              data: Array(20).fill(null), // Начальные значения данных для графика
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
      priceData(newPrice) {
        this.updateChartData(newPrice);
      }
    },
    mounted() {
      this.chart = new Chart(this.$refs.chart, {
        type: 'line',
        data: this.chartData,
        options: this.chartOptions
      });
    },
    methods: {
      updateChartData(newPrice) {
        // Добавляем новую цену в конец данных
        this.chartData.datasets[0].data.push(newPrice);
        this.chartData.labels.push(""); // добавляем пустую метку для X-оси
  
        // Ограничиваем количество точек на графике (например, до 20)
        if (this.chartData.datasets[0].data.length > 20) {
          this.chartData.datasets[0].data.shift();
          this.chartData.labels.shift();
        }
  
        // Обновляем график
        this.chart.update();
      }
    },
    beforeDestroy() {
      // Удаляем график при уничтожении компонента, чтобы избежать утечек памяти
      if (this.chart) {
        this.chart.destroy();
      }
    }
  };
  </script>
  
  <style scoped>
  canvas {
    height: 100px;
  }
  </style>
  