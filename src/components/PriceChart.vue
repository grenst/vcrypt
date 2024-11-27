<template>
  <div v-if="symbolName !== null">
    <p>Current price of {{ symbolName }}:</p>
    <div class="actual_price" :class="{ 'price-glow': isPriceChanged }">
      <span 
        v-for="(digit, index) in priceDigits" 
        :key="index" 
        :class="['price-digit', digitClasses[index]]">
        {{ digit }}
      </span>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    symbolName: {
      type: String,
      required: true
    },
    formattedPrice: {
      type: [String, Number],
      required: true
    }
  },
  data() {
    return {
      previousDigits: [],
      digitClasses: [], // Хранит классы анимации для каждой цифры
      previousPrice: null, // Хранит предыдущее значение цены
      isPriceChanged: false // Управляет эффектом свечения для всей цены
    };
  },
  computed: {
    priceDigits() {
      return String(this.formattedPrice).split('');
    }
  },
  watch: {
    formattedPrice(newPrice) {
      const newDigits = String(newPrice).split('');
      const direction = newPrice > this.previousPrice ? 'price-up' : 'price-down';

      // Применяем класс направления ко всем изменившимся цифрам
      this.digitClasses = newDigits.map((digit, index) => {
        if (this.previousDigits[index] && this.previousDigits[index] !== digit) {
          return direction;
        }
        return '';
      });

      // Активация эффекта свечения для всей цены
      this.isPriceChanged = true;

      // Устанавливаем таймеры для удаления классов анимации после 450 мс
      setTimeout(() => {
        this.digitClasses = new Array(newDigits.length).fill('');
        this.isPriceChanged = false; // Сброс свечения
      }, 350);

      // Обновляем previousDigits и previousPrice для сравнения при следующем изменении
      this.previousDigits = [...newDigits];
      this.previousPrice = newPrice;
    }
  }
};
</script>

<style scoped>
p {
  padding-top: 20px;
}
.actual_price {
  padding-top: 40px;
  font-family: 'Orbitron', sans-serif;
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.1em;
  font-weight: 600;
  font-size: 2.8em;
  color: white;
  transition: box-shadow 0.45s ease, opacity 0.45s ease;
}

/* Анимация для отдельных цифр */
.price-digit {
  display: inline-block;
  transition: transform 0.45s ease-in-out, opacity 0.45s ease;
}

/* Анимация прокрутки вверх (рост цены) */
.price-up {
  animation: scroll-up 0.85s forwards;
}

/* Анимация прокрутки вниз (падение цены) */
.price-down {
  animation: scroll-down 0.85s forwards;
}

/* Анимация прокрутки вверх */
@keyframes scroll-up {
  0% {
    transform: translateY(50%);
    opacity: 0;
    color: rgb(3, 190, 3);
  }
  50% {
    opacity: 1;
    transform: translateY(0);
  }
  100% {
    opacity: 0;
    transform: translateY(-100%);
  }
}

/* Анимация прокрутки вниз */
@keyframes scroll-down {
  0% {
    transform: translateY(-50%);
    opacity: 0;
    color: rgb(255, 0, 0);
  }
  50% {
    opacity: 1;
    transform: translateY(0);
  }
  100% {
    opacity: 0;
    transform: translateY(100%);
  }
}
</style>
