<template>
  <div class="container mx-auto flex flex-col items-center bg-gray-100 p-4">
    <div class="container">
      <section>
        <div class="flex">
          <div class="max-w-xs">
            <label for="wallet" class="block text-sm font-medium text-gray-700"
              >Тикер</label
            >
            <div class="mt-1 relative rounded-md shadow-md">
              <input
                v-model="ticker"
                @keyup="addError.state = false"
                type="text"
                name="wallet"
                id="wallet"
                class="block w-full pr-10 border-gray-300 text-gray-900 focus:outline-none focus:ring-gray-500 focus:border-gray-500 sm:text-sm rounded-md"
                placeholder="Например DOGE"
              />
            </div>
            <div
              class="flex bg-white shadow-md p-1 rounded-md shadow-md flex-wrap"
            >
              <span
                v-for="(item, idx) in filterAutoCompleate()"
                :key="idx"
                @click="addTicker(item)"
                class="inline-flex items-center px-2 m-1 rounded-md text-xs font-medium bg-gray-300 text-gray-800 cursor-pointer"
              >
                {{ item }}
              </span>
            </div>
            <div v-if="addError.state" class="text-sm text-red-600">
              {{ addError.text }}
            </div>
          </div>
        </div>
        <button
          type="button"
          @click="addTicker()"
          class="my-4 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
        >
          <!-- Heroicon name: solid/mail -->
          <svg
            class="-ml-0.5 mr-2 h-6 w-6"
            xmlns="http://www.w3.org/2000/svg"
            width="30"
            height="30"
            viewBox="0 0 24 24"
            fill="#ffffff"
          >
            <path
              d="M13 7h-2v4H7v2h4v4h2v-4h4v-2h-4V7zm-1-5C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 18c-4.41 0-8-3.59-8-8s3.59-8 8-8 8 3.59 8 8-3.59 8-8 8z"
            ></path>
          </svg>
          Добавить
        </button>
      </section>

      <template v-if="tickers.length > 0">
        <input
          v-model="filter"
          type="text"
          name="filterInput"
          id="filterInput"
          style="box-shadow: 0 0 10px rgba(0, 0, 0, 0.09)"
          class="block pr-10 border-gray-300 text-gray-900 focus:outline-none focus:ring-gray-500 focus:border-gray-500 sm:text-sm rounded-md"
          placeholder="Фильтр"
        />
        <hr class="w-full border-t border-gray-600 my-4" />
        <div class="mt-5 grid grid-cols-1 gap-5 sm:grid-cols-3">
          <div
            v-for="(item, key) in pageItems"
            :key="key"
            @click="selectTicker(item)"
            :class="{ 'border-4': selectedTicker === item }"
            class="bg-white overflow-hidden shadow rounded-lg border-purple-800 border-solid cursor-pointer"
          >
            <div class="px-4 py-5 sm:p-6 text-center">
              <dt class="text-sm font-medium text-gray-500 truncate">
                {{ item.name }} - USD
              </dt>
              <dd class="mt-1 text-3xl font-semibold text-gray-900">
                {{ item.price }}
              </dd>
            </div>
            <div class="w-full border-t border-gray-200"></div>
            <button
              class="flex items-center justify-center font-medium w-full bg-gray-100 px-4 py-4 sm:px-6 text-md text-gray-500 hover:text-gray-600 hover:bg-gray-200 hover:opacity-20 transition-all focus:outline-none"
              @click.stop="deleteTicker(item)"
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
                ></path></svg
              >Удалить
            </button>
          </div>
        </div>
        <hr class="w-full border-t border-gray-600 my-4" />
        <div class="paginate">
          <button
            type="button"
            class="my-4 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
            @click="page--"
            :disabled="page == 1"
          >
            Назад
          </button>
          <div class="paginate__items">
            <span
              class="item"
              v-for="(item, key) in renderPages()"
              :key="key"
              :class="{ clicked: item == page }"
              @click="page = item"
              >{{ item }}</span
            >
          </div>
          <button
            type="button"
            class="my-4 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
            @click="page++"
            :disabled="!hasNextPage"
          >
            Вперед
          </button>
        </div>
      </template>
      <section v-if="selectedTicker" class="relative">
        <h3 class="text-lg leading-6 font-medium text-gray-900 my-8">
          {{ selectedTicker.name }} - USD
        </h3>
        <div class="flex items-end border-gray-600 border-b border-l h-64">
          <div
            v-for="(item, idx) in normalizedGraph"
            :key="idx"
            :style="{ height: `${item}%` }"
            class="bg-purple-800 border w-10"
          ></div>
        </div>
        <button
          @click.stop="selectedTicker = null"
          type="button"
          class="absolute top-0 right-0"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            xmlns:xlink="http://www.w3.org/1999/xlink"
            xmlns:svgjs="http://svgjs.com/svgjs"
            version="1.1"
            width="30"
            height="30"
            x="0"
            y="0"
            viewBox="0 0 511.76 511.76"
            style="enable-background: new 0 0 512 512"
            xml:space="preserve"
          >
            <g>
              <path
                d="M436.896,74.869c-99.84-99.819-262.208-99.819-362.048,0c-99.797,99.819-99.797,262.229,0,362.048    c49.92,49.899,115.477,74.837,181.035,74.837s131.093-24.939,181.013-74.837C536.715,337.099,536.715,174.688,436.896,74.869z     M361.461,331.317c8.341,8.341,8.341,21.824,0,30.165c-4.16,4.16-9.621,6.251-15.083,6.251c-5.461,0-10.923-2.091-15.083-6.251    l-75.413-75.435l-75.392,75.413c-4.181,4.16-9.643,6.251-15.083,6.251c-5.461,0-10.923-2.091-15.083-6.251    c-8.341-8.341-8.341-21.845,0-30.165l75.392-75.413l-75.413-75.413c-8.341-8.341-8.341-21.845,0-30.165    c8.32-8.341,21.824-8.341,30.165,0l75.413,75.413l75.413-75.413c8.341-8.341,21.824-8.341,30.165,0    c8.341,8.32,8.341,21.824,0,30.165l-75.413,75.413L361.461,331.317z"
                fill="#718096"
                data-original="#000000"
              ></path>
            </g>
          </svg>
        </button>
      </section>
    </div>
  </div>
</template>

<script>
const AMOUNT_ELEMENTS = 1,
  PAGINATE_SPACE = 2;
export default {
  name: "App",
  data: () => ({
    ticker: "btc",
    tickers: [],
    selectedTicker: null,
    graph: [],
    autoCompleate: [],
    addError: {
      state: false,
      text: "",
    },
    page: 1,
    filter: "",
  }),

  computed: {
    startPage() {
      if (this.page - PAGINATE_SPACE <= 0) return 1;
      else if (this.page + PAGINATE_SPACE >= this.endPage) {
        let calcFirstPage = this.endPage - PAGINATE_SPACE * 2;
        return calcFirstPage > 0 ? calcFirstPage : 1;
      } else return this.page - PAGINATE_SPACE;
    },
    endPage() {
      if (Number(this.page) + PAGINATE_SPACE >= this.amountPages)
        return this.amountPages;
      else if (Number(this.page) - PAGINATE_SPACE <= 0) {
        let calcEndPage =
          Number(this.page) +
          PAGINATE_SPACE +
          Math.abs(this.page - PAGINATE_SPACE) +
          1;
        return calcEndPage > this.amountPages ? this.amountPages : calcEndPage;
      } else return Number(this.page) + PAGINATE_SPACE;
    },

    startPosition() {
      return (this.page - 1) * AMOUNT_ELEMENTS;
    },

    endPosition() {
      return this.page * AMOUNT_ELEMENTS;
    },

    hasNextPage() {
      return this.filteredTickers.length > this.endPosition;
    },

    amountPages() {
      return Math.ceil(this.filteredTickers.length / AMOUNT_ELEMENTS);
    },

    filteredTickers() {
      return this.tickers.filter((ticker) =>
        ticker.name.includes(this.filter.toUpperCase())
      );
    },

    pageItems() {
      return this.filteredTickers.slice(this.startPosition, this.endPosition);
    },

    normalizedGraph() {
      const MAX_VALUE = Math.max(...this.graph);
      const MIN_VALUE =
        Math.min(...this.graph) != MAX_VALUE ? Math.min(...this.graph) : 0;
      return this.graph.map(
        (price) => 5 + ((price - MIN_VALUE) * 95) / (MAX_VALUE - MIN_VALUE)
      );
    },

    pageStateOptions(){
      return {
        page: this.page,
        filter: this.filter
      };
    }
    
  },

  methods: {
    subscribeToUpdate(tickerName) {
      setInterval(async () => {
        let data = await fetch(
          `https://min-api.cryptocompare.com/data/price?fsym=${tickerName}&tsyms=USD&api_key=dc023682b313b759cd8c24fcc86f57e61f8d0dc95bc019fbe5aa59c7791d0ef7`
        );
        data = await data.json();
        if (data.Response != "Error")
          this.tickers.find((item) => item.name === tickerName).price =
            data.USD > 1 ? data.USD.toFixed(2) : data.USD.toPrecision(2);
        if (this.selectedTicker?.name === tickerName && this.selectedTicker?.price != this.graph[this.graph.length-1]) this.graph.push(data.USD);
      }, 5000);
    },

    addTicker(targetTicker = null) {
      if (this.tickers.find((item) => item.name == this.ticker.toUpperCase())) {
        this.addError.text = "Такой тикер уже добавлен";
        this.addError.state = true;
        return;
      } else if (this.ticker == "" && targetTicker == null) {
        this.addError.text = "Невозможно добавить тикер без названия";
        this.addError.state = true;
        return;
      }
      const CURRENT_TICKER = {
        name: targetTicker == null ? this.ticker.toUpperCase() : targetTicker,
        price: 0,
      };

      this.subscribeToUpdate(CURRENT_TICKER.name);
      this.ticker = "";
      this.tickers.push(CURRENT_TICKER);
      this.filter = "";
    },

    renderPages() {
      let range = [];
      for (let i = this.startPage; i <= this.endPage; i++) range.push(i);
      return range;
    },

    selectTicker(ticker) {
      this.selectedTicker = ticker;
    },

    deleteTicker(elem) {
      this.tickers = this.tickers.filter((t) => t != elem);
      if(this.selectedTicker === elem)
        this.selectedTicker = null;
    },

    filterAutoCompleate() {
      if (this.ticker.length != 0) {
        return this.autoCompleate
          .filter((item) => item.indexOf(this.ticker.toUpperCase()) >= 0)
          .splice(0, 4);
      } else {
        return this.autoCompleate.splice(0, 4);
      }
    },
  },

  watch: {
    tickers(){
      localStorage.setItem("cryptonomicon-list", JSON.stringify(this.tickers));
    },

    selectedTicker(){
        this.graph = [];
    },

    pageItems(){
      if(this.pageItems.length === 0 && this.page > 1)
        this.page = this.amountPages;
    },  
    
    filter() {
      this.page = 1;
    },
    pageStateOptions() {
      let currentPath = `${window.location.pathname}?`;
      for(let key in this.pageStateOptions)
        currentPath += `${key}=${this.pageStateOptions[key]}&`;
      currentPath = currentPath.slice(0, -1);
      window.history.pushState(
        null,
        document.title,
        currentPath
      );
    },
  },

  async created() {
    let data = await fetch(
      `https://min-api.cryptocompare.com/data/blockchain/list?api_key=dc023682b313b759cd8c24fcc86f57e61f8d0dc95bc019fbe5aa59c7791d0ef7`
    );
    data = await data.json();
    const LOADED_AUTO_COMPLEATE = [];
    for (let key in data.Data) {
      LOADED_AUTO_COMPLEATE.push(key);
    }
    this.autoCompleate = LOADED_AUTO_COMPLEATE;

    const tickersData = localStorage.getItem("cryptonomicon-list");
    if (tickersData) {
      this.tickers = JSON.parse(tickersData).map((ticker) => ({
        name: ticker.name,
        price: 0,
      }));
      // this.tickers.forEach(ticker => this.subscribeToUpdate(ticker.name));
    }

    const windowData = Object.fromEntries(
      new URL(window.location).searchParams.entries()
    );
    if (Object.keys(windowData).length) {
      this.filter = windowData.filter;
      this.page = windowData.page;
    }
  },
};
</script>
<style src="./app.css"></style>
