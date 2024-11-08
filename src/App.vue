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
        <PriceChart :price="price" />
      </div>
    </div>
  </div>
</template>

<script>
import PriceChart from './PriceChart.vue';

export default {
  components: {
    PriceChart,
  },
  data() {
    return {
      symbol: "",
      price: null,
      ws: null,
    };
  },
  computed: {
    symbolName() {
      return this.realSymbol + "USDT";
    },
    realSymbol() {
      return this.symbol.trim().toUpperCase();
    },
    formattedPrice() {
      const decimalPlaces = this.getDecimalPlaces(this.symbolName);
      return this.price !== null ? this.price.toFixed(decimalPlaces) : null;
    },
  },
  methods: {
    connectToWebSocket(symbol) {
      if (this.ws) {
        this.ws.close();
      }

      this.ws = new WebSocket(`wss://stream.binance.com:9443/ws/${symbol.toLowerCase()}@ticker`);

      this.ws.onmessage = (event) => {
        const data = JSON.parse(event.data);
        this.price = parseFloat(data.c);
      };

      this.ws.onerror = (error) => {
        console.error('WebSocket error:', error);
      };

      this.ws.onclose = () => {
        console.log('WebSocket closed');
      };
    },
    getDecimalPlaces(symbol) {
      const precisionMap = {
        BTCUSDT: 2,
        ETHUSDT: 2,
        LTCUSDT: 2,
        XRPUSDT: 4,
        DOGEUSDT: 6,
      };
      return precisionMap[symbol] || 2;
    },
  },
  beforeDestroy() {
    if (this.ws) {
      this.ws.close();
    }
  },
};
</script>

<style scoped>
.wrapper {
  width: 400px;
  height: 400px;
  border: 3px solid rgb(47, 97, 84);
  border-radius: 50px;
}
</style>
