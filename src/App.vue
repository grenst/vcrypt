<template>
  <div class="p-6 bg-gray-800 shadow rounded-lg">
    <div class="wrapper">
      <h1 class="text-xl border-b">Database Checker</h1>
      <p>Checkout symbol <b>{{ realSymbol == "" ? "in database" : symbolName }}</b></p>
      <input class="test" v-model="symbol" placeholder="Enter symbol "></input>
      <button v-if="realSymbol" class="m-2 p-2 text-black font-medium rounded-lg transition ease-in-out delay-50 bg-cyan-500 shadow-lg hover:text-white hover:-translate-y-0.3 hover:bg-indigo-500 duration-150 shadow-indigo-500/50 hover:shadow-cyan-500/50" @click="connectToWebSocket(symbolName)">
        Submit
      </button>
      <div v-if="realSymbol && price !== null">
        <p>Current price of {{ symbolName }}: {{ formattedPrice }}</p>
        <PriceChart :priceData="priceHistory" />
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import PriceChart from '/src/components/PriceChart.vue';

export default {
  components: {
    PriceChart
  },
  data() {
    return {
      symbol: "",
      price: null,
      ws: null, // WebSocket connection
      priceHistory: [] // Array to store price history for chart
    }
  },
  computed: {
    symbolName() {
      return this.realSymbol + "USDT";
    },
    realSymbol() {
      return this.symbol.trim().toUpperCase();
    },
    // Форматирование цены в зависимости от символа
    formattedPrice() {
      const decimalPlaces = this.getDecimalPlaces(this.symbolName);
      return this.price !== null ? this.price.toFixed(decimalPlaces) : null;
    }
  },
  methods: {
    // Подключение к WebSocket для получения обновлений цены
    connectToWebSocket(symbol) {
      // Закрываем предыдущее соединение, если оно существует
      if (this.ws) {
        this.ws.close();
      }

      // Создаем новое WebSocket соединение
      this.ws = new WebSocket(`wss://stream.binance.com:9443/ws/${symbol.toLowerCase()}@ticker`);

      // Обрабатываем сообщение от WebSocket
      this.ws.onmessage = (event) => {
        const data = JSON.parse(event.data);
        this.price = parseFloat(data.c); // 'c' - текущая цена в сообщении от Binance
        this.updatePriceHistory(this.price);
      };

      // Обрабатываем ошибки WebSocket
      this.ws.onerror = (error) => {
        console.error('WebSocket error:', error);
      };

      // Закрываем соединение при размонтировании компонента
      this.ws.onclose = () => {
        console.log('WebSocket closed');
      };
    },
    // Добавление новой цены в массив priceHistory и ограничение его длины
    updatePriceHistory(newPrice) {
      this.priceHistory.push(newPrice);
      if (this.priceHistory.length > 20) { // Ограничиваем количество точек, например, 20
        this.priceHistory.shift();
      }
    },
    // Определение количества знаков после запятой для каждого символа
    getDecimalPlaces(symbol) {
      const precisionMap = {
        BTCUSDT: 2,
        ETHUSDT: 2,
        LTCUSDT: 2,
        XRPUSDT: 4,
        DOGEUSDT: 6,
        // Добавьте другие символы, если нужно
      };
      return precisionMap[symbol] || 2; // значение по умолчанию - 2 знака
    }
  },
  beforeDestroy() {
    // Закрываем WebSocket соединение при уничтожении компонента
    if (this.ws) {
      this.ws.close();
    }
  }
}
</script>

<style scoped>
.wrapper {
  width: 400px;
  height: 300px;
  border: 3px solid rgb(47, 97, 84);
  border-radius: 50px;
}
.wrapper h1 {
  margin-top: 50px;
}

.wrapper input {
  margin-top: 30px;
  padding: 5px 8px;
  background: transparent;
  border: 0;
  border-bottom: 2px solid #110813;
  font-size: 0.9em;
  outline: none;
}

.wrapper input:focus {
  border-bottom-color: #6e2d7d;
}

button {
  outline: none;
}
</style>
