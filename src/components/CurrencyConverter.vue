<template>
  <div class="converter">
    <div class="converter-container">
      <!-- Converter text -->
      <p class="converter-container__error-message" v-if="errorMessage" v-text="errorMessage"></p>
      <div class="converter-container__text">
        <p class="converter-container__text--small" v-text="startTextLabel"></p>
        <p class="converter-container__text--large" v-text="convertedTextLabel"></p>
        <p class="converter-container__text--small" v-html="todayDate"></p>
      </div>
      <!-- Converter inputs -->
      <div class="converter-container__inputs">
        <div class="converter-container__inputs__first-group form-group">
          <input type="number" id="converter-first-input" :value="startValue" @input="calculateConversion">
          <select id="converter-first-select" ref="firstSelect" @change="calculateConversion" v-model="firstSelect">
            <option v-for="option in currency" :value="option.abbreviation" v-text="option.name"></option>
          </select>
        </div>
        <div class="converter-container__inputs__second-group">
          <input type="number" id="converter-second-input" :value="convertedValue" @input="calculateConversion">
          <select id="converter-second-select" ref="secondSelect" @change="calculateConversion" v-model="secondSelect">
            <option v-for="option in currency" :value="option.abbreviation" v-text="option.name"></option>
          </select>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import currencyConverter from "@/components/classes/currencyConverter";

export default {
  name: "CurrencyConverter",

  data() {
    const currency = currencyConverter.currency;
    return {
      startValue: 1,
      convertedValue: 0,
      startTextLabel: "",
      convertedTextLabel: "",
      currency,
      firstSelect: "EUR",
      secondSelect: "USD",
      errorMessage: false
    }
  },

  mounted() {
    this.calculateConversion();
  },

  methods: {
    async calculateConversion(el) {
      this.errorMessage = false;
      const query = this.firstSelect + '_' + this.secondSelect;
      const url = "https://free.currconv.com/api/v7/convert?q=" + query + "&compact=ultra&apiKey=0fa15aba08a788864288";

      try {
        const response = await this.axios.get(url);
        const exchangeRate = response.data[query];

        //calculate exchange rate
        if (!el) {
          this.convertedValue = (exchangeRate * 1).toFixed(4);
        } else {
          const firstInput = document.getElementById("converter-first-input").value.replace(',', '.');
          const secondInput = document.getElementById("converter-second-input").value.replace(',', '.');
          if (el.target.id === "converter-second-input") {
            this.startValue = (exchangeRate * secondInput).toFixed(4);
            this.convertedValue = secondInput;
          } else {
            this.startValue = firstInput;
            this.convertedValue = (exchangeRate * firstInput).toFixed(4);
          }
        }

        this.startTextLabel = `1 ${this.$refs.firstSelect.selectedOptions[0].text} ${currencyConverter.labels.equal}`;
        this.convertedTextLabel = exchangeRate.toFixed(4) + " " + this.$refs.secondSelect.selectedOptions[0].text;
      } catch (error) {
        this.errorMessage = error.response.data.error;
      }

    }
  },

  watch: {
    firstSelect(val, oldVal) {
      if (this.secondSelect === val) {
        this.firstSelect = val;
        this.secondSelect = oldVal;
      }
    },
    secondSelect(val, oldVal) {
      if (this.firstSelect === val) {
        this.firstSelect = oldVal;
        this.secondSelect = val;
      }
    }
  },

  computed: {
    todayDate() {
      const date = new Date();
      const today = Date.UTC(date.getUTCFullYear(), date.getUTCMonth(), date.getUTCDate(),
          date.getUTCHours(), date.getUTCMinutes());
      return new Date(today).toUTCString().replace(/:[^:]*$/, '') + ' UTC';
    }
  }
}
</script>

<style lang="scss" scoped>
@import "../sass/currencycalculator";
</style>