<template>
  <div class="container mx-auto flex flex-col items-center bg-gray-100 p-4">
    <div class="container">
      <AddTicker @add-ticker="add" />
      <template v-if="tickers.length">
        <hr class="w-full border-t border-gray-600 my-4" />
        <button
          v-if="startIndex > 0"
          @click="page -= 1"
          class="mx-2 my-4 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
        >
          Back
        </button>
        <button
          v-if="endIndex < tickers.length"
          @click="page += 1"
          class="mx-2 my-4 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
        >
          Next
        </button>
        <div>filter: <input v-model="filter" /></div>

        <hr class="w-full border-t border-gray-600 my-4" />

        <dl class="mt-5 grid grid-cols-1 gap-5 sm:grid-cols-3">
          <div
            v-for="tick of paginatedTickers"
            :key="tick.name"
            @click="select(tick)"
            :class="{
              'border-4': selectedTicker === tick,
              'bg-red-200': tick.value === undefined,
            }"
            class="overflow-hidden shadow rounded-lg border-purple-800 border-solid cursor-pointer"
          >
            <div class="px-4 py-5 sm:p-6 text-center">
              <dt class="text-sm font-medium text-gray-500 truncate">
                {{ tick.name }}
              </dt>
              <dd class="mt-1 text-3xl font-semibold text-gray-900">
                {{ tick.value }}
              </dd>
            </div>
            <div class="w-full border-t border-gray-200"></div>
            <button
              @click.stop="dell(tick)"
              :class="{
                'bg-gray-100': tick.value !== undefined,
                'bg-red-200': tick.value === undefined,
                'hover:bg-gray-200': tick.value !== undefined,
                'hover:bg-red-300': tick.value === undefined,
              }"
              class="flex items-center justify-center font-medium w-full bg-gray-100 px-4 py-4 sm:px-6 text-md text-gray-500 hover:text-gray-600 hover:bg-gray-200 hover:opacity-20 transition-all focus:outline-none"
            >
              <svg
                class="h-5 w-5"
                xmlns="http://www.w3.org/2000/svg"
                viewBox="0 0 20 20"
                fill="#718096"
                aria-hidden="true"
              >
                <path
                  fill-rule="evenodd"
                  d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z"
                  clip-rule="evenodd"
                ></path>
              </svg>
              Удалить
            </button>
          </div>
        </dl>
        <hr class="w-full border-t border-gray-600 my-4" />
      </template>
      <DrawingGraph
        :selected-ticker="selectedTicker"
        :graph-of-ticker="graph"
      />
    </div>
  </div>
</template>

<script>
import AddTicker from "./componets/AddTicker.vue";
import DrawingGraph from "./componets/DrawGraph.vue";

export default {
  name: "App",
  components: {
    AddTicker,
    DrawingGraph,
  },
  data() {
    return {
      selectedTicker: null,
      tickers: [],
      graph: [],
      updateInterval: null,
      page: 1,
      maxGraphElements: 1,
      filter: "",
    };
  },

  computed: {
    startIndex() {
      return (this.page - 1) * 6;
    },

    endIndex() {
      return this.page * 6;
    },

    filteredTickers() {
      return this.tickers.filter((ticker) =>
        ticker.name.includes(this.filter.toUpperCase())
      );
    },

    paginatedTickers() {
      return this.filteredTickers.slice(this.startIndex, this.endIndex);
    },
  },
  watch: {
    graph(newGraph) {
      if (newGraph.length > 0) {
        this.calculateMaxGraphElements();
      }
    },
    paginatedTickers() {
      if (this.paginatedTickers.length === 0 && this.page > 1) {
        this.page -= 1;
      }
    },
  },

  mounted() {
    this.restoreState();
    this.calculateMaxGraphElements(); // Initial calculation
    window.addEventListener("resize", this.calculateMaxGraphElements);
    if (this.tickers.length > 0 && !this.updateInterval) {
      this.updateInterval = setInterval(this.updateTickers, 2000);
    }
  },

  beforeUnmount() {
    window.removeEventListener("resize", this.calculateMaxGraphElements);
    if (this.updateInterval) {
      clearInterval(this.updateInterval);
    }
  },

  methods: {
    add(ticker) {
      if (this.tickers.some((t) => t.name === ticker)) {
        alert("Ticker already added");
        return;
      }

      const newItem = { name: ticker.toUpperCase(), value: "-" };
      this.tickers.push(newItem);

      localStorage.setItem("cryptonimicon-list", JSON.stringify(this.tickers));
      this.filter = "";

      if (!this.updateInterval) {
        this.updateInterval = setInterval(this.updateTickers, 2000);
      }
    },
    restoreState() {
      const tickersData = localStorage.getItem("cryptonimicon-list");
      if (tickersData) {
        this.tickers = JSON.parse(tickersData);
      }

      if (this.tickers.length > 0 && !this.updateInterval) {
        this.updateInterval = setInterval(this.updateTickers, 2000);
      }
    },

    async updateTickers() {
      if (this.tickers.length === 0) {
        if (this.updateInterval) {
          clearInterval(this.updateInterval);
          this.updateInterval = null;
        }
        return;
      }

      try {
        for (const ticker of this.tickers) {
          const response = await fetch(
            `https://min-api.cryptocompare.com/data/price?fsym=${ticker.name}&tsyms=USD&api_key=9384a78ce72773fbcea39ffc4a0f7e5c06488b4d220f2bf1677a35fc07d5f879`
          );

          if (!response.ok) {
            throw new Error(`Error fetching data for ticker ${ticker.name}`);
          }

          const data = await response.json();
          ticker.value = data.USD;

          if (this.selectedTicker && this.selectedTicker.name === ticker.name) {
            this.graph.push(data.USD);
            // Ensure the graph does not exceed maxGraphElements
            while (this.graph.length > this.maxGraphElements) {
              this.graph.shift();
            }
          }
        }
      } catch (error) {
        console.error("Error updating tickers:", error);
      }
    },

    select(ticker) {
      this.selectedTicker = ticker;
      this.graph = [];
      this.$nextTick(() => {
        this.calculateMaxGraphElements();
      });
    },

    calculateMaxGraphElements() {
      const graphContainerWidth = window.innerWidth;
      const elementWidth = 38;
      this.maxGraphElements = Math.floor(graphContainerWidth / elementWidth);
    },

    dell(deltick) {
      this.tickers = this.tickers.filter((t) => t.name !== deltick.name);

      if (this.selectedTicker && this.selectedTicker.name === deltick.name) {
        this.selectedTicker = null;
        this.graph = [];
      }

      localStorage.setItem("cryptonimicon-list", JSON.stringify(this.tickers));

      if (this.tickers.length === 0 && this.updateInterval) {
        clearInterval(this.updateInterval);
        this.updateInterval = null;
      }
    },
  },
};
</script>
