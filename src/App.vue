<template>
  <div class="p-6 bg-neutral-800 shadow rounded-3xl">
    <div class="wrapper">
      <h1 class="text-xl border-b">Database Checker</h1>
      <p>Checkout symbol <b>{{ realSymbol == "" ? "in database" : symbolName }}</b></p>
      <input class="test" v-model="symbol" placeholder="Enter symbol"></input>
      <button v-if="realSymbol" 
              class="m-2 p-2 text-black font-medium rounded-lg transition ease-in-out delay-50 bg-cyan-500 shadow-lg hover:text-white hover:-translate-y-0.3 hover:bg-indigo-500 duration-150 shadow-indigo-500/50 hover:shadow-cyan-500/50" 
              @click="checkSymbolAndConnect">
        Submit
      </button>
      <PriceChart v-if="realSymbol && price !== null" 
                  :symbolName="symbolName" 
                  :formattedPrice="formattedPrice" />
      <p v-if="error" class="text-red-500 mt-2">{{ error }}</p>
    </div>
  </div>
</template>

<script>
import PriceChart from './components/PriceChart.vue';

export default {
  components: {
    PriceChart
  },
  data() {
    return {
      symbol: "",
      price: null,
      ws: null,
      error: null,
      symbolPrecisions: {},
    }
  },
  computed: {
    symbolName() {
      return this.realSymbol + "USDT";
    },
    realSymbol() {
      return this.symbol.trim().toUpperCase();
    },
    formattedPrice() {
      const decimalPlaces = this.symbolPrecisions[this.symbolName] ?? 2;
      return this.price !== null ? this.price.toFixed(decimalPlaces) : null;
    }
  },
  methods: {
    async fetchSymbolPrecision(symbol) {
      try {
        const response = await fetch(`https://api.binance.com/api/v3/exchangeInfo?symbol=${symbol}`);
        const data = await response.json();
        
        if (data.symbols && data.symbols[0]) {
          // Находим фильтр с ценой
          const priceFilter = data.symbols[0].filters.find(
            filter => filter.filterType === "PRICE_FILTER"
          );
          
          if (priceFilter) {
            // Получаем количество десятичных знаков из tickSize
            const tickSize = priceFilter.tickSize;
            const decimalPlaces = this.getDecimalPlacesFromTickSize(tickSize);
            this.symbolPrecisions[symbol] = decimalPlaces;
            return decimalPlaces;
          }
        }
        throw new Error("Could not determine price precision");
      } catch (err) {
        console.error("Error fetching symbol precision:", err);
        return 2; // Возвращаем значение по умолчанию в случае ошибки
      }
    },
    
    getDecimalPlacesFromTickSize(tickSize) {
      // Преобразуем tickSize в строку и убираем научную нотацию
      const strTickSize = parseFloat(tickSize).toString();
      // Находим позицию десятичной точки
      const decimalPos = strTickSize.indexOf('.');
      if (decimalPos === -1) return 0;
      
      // Считаем значащие нули после десятичной точки
      let zeros = 0;
      for (let i = decimalPos + 1; i < strTickSize.length; i++) {
        if (strTickSize[i] === '0') {
          zeros++;
        } else {
          break;
        }
      }
      return zeros + 1;
    },

    async checkSymbolAndConnect() {
      this.error = null;
      try {
        if (!this.symbolPrecisions[this.symbolName]) {
          await this.fetchSymbolPrecision(this.symbolName);
        }
        this.connectToWebSocket(this.symbolName);
      } catch (err) {
        this.error = "Invalid symbol or network error";
        console.error(err);
      }
    },

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
        this.error = "WebSocket connection error";
      };
      
      this.ws.onclose = () => {
        console.log('WebSocket closed');
      };
    }
  },
  beforeDestroy() {
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
  border: 1px solid rgb(47, 97, 84); 
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