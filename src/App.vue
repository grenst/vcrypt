<template>
  <div className="p-6 bg-gray-800 shadow rounded-lg">
    <div class="wrapper">
      <h1 className="text-xl border-b">Database Checker</h1>
      <p>Checkout symbol <b>{{ realSymbol == "" ? "in database" : symbolName }}</b></p>
      <input class="test" v-model="symbol" placeholder="Enter symbol "></input>
      <button v-if="!realSymbol == ''" className="m-2 p-2 text-black font-medium rounded-lg transition ease-in-out delay-50 bg-cyan-500 shadow-lg hover:text-white hover:-translate-y-0.3 hover:bg-indigo-500 duration-150 shadow-indigo-500/50 hover:shadow-cyan-500/50" @click="fetchPrice(symbolName)">
        Submit
      </button>
      <div v-if="!realSymbol == '' & price != null">
        <p>Current price of BTC/USDT: {{ price }}</p>
      </div>
      <div v-else="priceNull()"></div>
    </div>
  </div>
</template>


<script>
import axios from 'axios';

export default {
  data() {
    return {
      symbol: "",
      price: null
    }
  },
  computed: {
    symbolName() {
      return this.realSymbol + "USDT"
    },
    realSymbol() {
      return this.symbol.trim().toUpperCase()
    }
  },
  methods: {
    async fetchPrice(symbol) {
      try {
        const response = await axios.get(`https://api.binance.com/api/v3/ticker/price?symbol=${symbol}`);
        this.price = Math.round(response.data.price);
      } catch (error) {
        console.error('Error fetching price:', error);
      }
    }
  },
  priceNull() {
    price = null
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

.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
button {
  outline: none;
}
</style>
