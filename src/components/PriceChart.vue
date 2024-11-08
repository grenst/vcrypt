<!-- PriceChart.vue -->
<template>
    <div>
      <canvas ref="chart"></canvas>
    </div>
  </template>
  
  <script>
  import { Chart, registerables } from 'chart.js';
  Chart.register(...registerables);
  
  export default {
    props: ['price'],
    data() {
      return {
        chart: null,
        prices: [], // массив для хранения последних цен
        maxPoints: 20 // максимальное количество точек на графике
      };
    },
    watch: {
      price(newPrice) {
        if (newPrice !== null) {
          this.updateChart(newPrice);
        }
      }
    },
    mounted() {
      this.initChart();
    },
    methods: {
      initChart() {
        this.chart = new Chart(this.$refs.chart, {
          type: 'line',
          data: {
            labels: Array(this.maxPoints).fill(''), // пустые подписи на оси X
            datasets: [
              {
                label: 'Price',
                data: [],
                borderColor: 'rgba(75, 192, 192, 1)',
                borderWidth: 2,
                fill: false,
                tension: 0.2,
              },
            ],
          },
          options: {
            responsive: true,
            maintainAspectRatio: false,
            scales: {
              x: { display: false }, // скрываем ось X
              y: { display: true },
            },
          },
        });
      },
      updateChart(newPrice) {
        if (this.prices.length >= this.maxPoints) {
          this.prices.shift(); // удаляем старую цену, если превышено maxPoints
          this.chart.data.labels.shift();
        }
  
        // Добавляем новую цену и обновляем график
        this.prices.push(newPrice);
        this.chart.data.labels.push(''); // добавляем пустую метку на оси X
        this.chart.data.datasets[0].data = this.prices;
        this.chart.update();
      },
    },
  };
  </script>
  
  <style scoped>
  div {
    height: 150px;
  }
  </style>
  