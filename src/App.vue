<template>
  <div class="p-6 bg-neutral-900 shadow rounded-3xl">
    <div class="wrapper">
      <div class="overlay"></div>

      <div class="word" ref="titleText">
        <h1 class="text-head border-b border-green-600">Database Checker</h1>
      </div>
      <div class="word" ref="descriptionText">
        <p class="text-check">Checkout symbol <b>{{ realSymbol == "" ? "in database" : symbolName }}</b></p>
      </div>

      <div class="input-group">
        <input v-model="symbol" placeholder="Enter symbol" />
        <button v-if="realSymbol" ref="buttonText" 
                class="m-2 text-black rounded-lg transition ease-in-out delay-50 bg-green-600 shadow-lg hover:text-white hover:-translate-y-0.3 hover:bg-green-500 duration-150 shadow-green-600/50 hover:shadow-green-500/50" 
                @click="checkSymbolAndConnect">
          <img src="../src/assets/enter.png" alt="Enter" class="button-icon"/>
        </button>
      </div>

      <PriceChart v-if="realSymbol && price !== null" :symbolName="symbolName" :formattedPrice="formattedPrice" />
      <div class="word" v-if="error" ref="errorText">
        <p class="text-red-500 mt-2">{{ error }}</p>
      </div>
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
      chars: 'ABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890!@#$%^&*()-_=+{}|[]\\;\':"<>?,./`~'.split('')
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
      const decimalPlaces = this.symbolPrecisions[this.symbolName] ?? 2;
      return this.price !== null ? this.price.toFixed(decimalPlaces) : null;
    }
  },
  mounted() {
    this.startTickerEffect(this.$refs.titleText);
    this.startTickerEffect(this.$refs.descriptionText);
    this.startTickerEffect(this.$refs.buttonText);
    if (this.error) this.startTickerEffect(this.$refs.errorText);
  },
  methods: {
    async fetchSymbolPrecision(symbol) {
      try {
        const response = await fetch(`https://api.binance.com/api/v3/exchangeInfo?symbol=${symbol}`);
        const data = await response.json();

        if (data.symbols && data.symbols[0]) {
          const priceFilter = data.symbols[0].filters.find(
            filter => filter.filterType === "PRICE_FILTER"
          );
          if (priceFilter) {
            const tickSize = priceFilter.tickSize;
            const decimalPlaces = this.getDecimalPlacesFromTickSize(tickSize);
            this.symbolPrecisions[symbol] = decimalPlaces;
            return decimalPlaces;
          }
        }
        throw new Error("Could not determine price precision");
      } catch (err) {
        console.error("Error fetching symbol precision:", err);
        return 2;
      }
    },
    
    getDecimalPlacesFromTickSize(tickSize) {
      const strTickSize = parseFloat(tickSize).toString();
      const decimalPos = strTickSize.indexOf('.');
      if (decimalPos === -1) return 0;

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
    },

    startTickerEffect(element) {
      const letters = Array.from(element.innerText);
      let originalText = element.innerText;
      let letterIndex = 0;
      let cycleCount = 5;
      let cycleCurrent = 0;
      let done = false;

      const loop = () => {
        if (cycleCurrent < cycleCount) {
          cycleCurrent++;
          element.innerText = letters
            .map((char, index) =>
              index <= letterIndex ? char : this.chars[Math.floor(Math.random() * this.chars.length)]
            )
            .join('');
        } else if (letterIndex < letters.length) {
          element.innerText = originalText.slice(0, letterIndex + 1) + originalText.slice(letterIndex + 1).replace(/./g, () => this.chars[Math.floor(Math.random() * this.chars.length)]);
          cycleCurrent = 0;
          letterIndex++;
        } else {
          done = true;
        }
        
        if (!done) {
          requestAnimationFrame(loop);
        }
      };

      loop();
    }
  },
  beforeDestroy() {
    if (this.ws) {
      this.ws.close();
    }
  }
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css?family=Source+Code+Pro:400');

.wrapper {
  padding-top: 15px;
  width: 400px;
  height: 300px;
  padding-inline: 10px;
  /* border: 1px solid rgb(47, 97, 84);  */
  border-radius: 50px;
  font-family: 'Source Code Pro', monospace;
  background: radial-gradient(#222922, #000500);
}

.word {
  font-family: 'Source Code Pro', monospace;
  color: #25ff99;
  font-size: 1.2em;
  line-height: 1.2em;
  text-shadow: 0 0 10px rgba(50, 255, 50, 0.5), 0 0 5px rgba(100, 255, 100, 0.5);
  position: relative;
}

.word .text-head {
  font-size: 1.8em;
}

.word .text-check {
  font-size: 1em;
}

.overlay {
  background-image: linear-gradient(transparent 0%, rgba(10, 16, 10, 0.5) 50%);
  background-size: 1000px 2px;
  bottom: 0;
  left: 0;
  position: absolute;
  right: 0;
  top: 0;
  z-index: 1;
}

p {
  font-size: 0.8em;
}

.input-group {
  display: flex;
  align-items: center;
  gap: 8px; /* Промежуток между input и button */
}

.wrapper input {
  flex: 1; /* Занимает оставшееся пространство */
  color: #098b4e;
  margin-top: 30px;
  padding: 5px 8px;
  background: transparent;
  border: 0;
  border-bottom: 1px solid #224725;
  font-size: 1.2em;
  outline: none;
  position: relative;
  z-index: 2;
}

.wrapper input:focus {
  border-bottom-color: 2px solid #2d7d36;
}

button {
  margin-top: 45px;
  display: flex;
  align-items: center;
  justify-content: center;
  max-height: 30px;
  max-width: 50px;
  outline: none;
  font-size: 0.9em;
  font-weight: 700;
  display: inline-flex;
  position: relative;
  z-index: 2;
}
</style>