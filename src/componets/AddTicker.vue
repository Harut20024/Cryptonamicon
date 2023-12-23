<template>
  <section>
    <div class="flex">
      <div class="max-w-xs">
        <label for="wallet" class="block text-sm font-medium text-gray-700"
          >Тикер</label
        >
        <div class="mt-1 relative rounded-md shadow-md">
          <input
            v-model="ticker"
            @keydown.enter="add()"
            type="text"
            name="wallet"
            id="wallet"
            class="block w-full pr-10 border-gray-300 text-gray-900 focus:outline-none focus:ring-gray-500 focus:border-gray-500 sm:text-sm rounded-md"
            placeholder="Например DOGE"
          />
        </div>
        <FinderCoins :tickersName="ticker" @update-ticker="ticker = $event" />
      </div>
    </div>
    <AddButton />
  </section>
</template>

<script>

import AddButton from "./PlusSign.vue";
import FinderCoins from "./FinderCoins.vue";


export default {
  components: {
    AddButton,
    FinderCoins,
  },
  data() {
    return {
      ticker: "",
    };
  },
  emits: {
    "add-ticker": (value) => typeof value === "string",
  },
  methods: {
    add() {
      if (this.ticker.length < 3) {
        alert("you enter not rigth");
        this.ticker = "";
        return;
      }
      this.$emit("add-ticker", this.ticker);
      this.ticker = "";
    },
  },
};
</script>
