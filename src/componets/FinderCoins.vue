<template>
  <div class="flex mt-1">
    <div v-for="coin in filteredCoins" :key="coin.Symbol" class="mr-1">
      <button
        @click="emitTickerUpdate(coin.Symbol)"
        class="my-1 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
      >
        {{ coin.Symbol }}
      </button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    tickersName: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      coinSymbols: [],
    };
  },
  mounted() {
    this.fetchCoins();
  },
  computed: {
    filteredCoins() {
      if (!this.tickersName) return [];
      return this.coinSymbols
        .filter((coin) =>
          coin.Symbol.toUpperCase().includes(this.tickersName.toUpperCase())
        )
        .slice(0, 6);
    },
  },
  methods: {
    emitTickerUpdate(ticker) {
      this.$emit("update-ticker", ticker);
    },
    async fetchCoins() {
      try {
        const response = await fetch(
          "https://min-api.cryptocompare.com/data/all/coinlist?summary=true"
        );
        const data = await response.json();
        if (data.Response === "Success" && data.Data) {
          this.coinSymbols = Object.values(data.Data);
        }
      } catch (error) {
        console.error("Error fetching coins:", error);
      }
    },
  },
};
</script>
