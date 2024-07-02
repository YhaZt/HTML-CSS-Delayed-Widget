<template>
  <div :class="[darkMode ? 'dark' : 'light', 'container', 'mx-auto', 'p-4']">
    <div class="flex justify-between items-center mb-4" >
      <select v-model="currentMarketId" @change="fetchData" class="border p-2 rounded" :style="{ color: darkMode ? 'var(--text-color)' : '' }">
        <option value="0">SGX</option>
        <option value="2">Bursa</option>
        <option value="3">Nasdaq</option>
      </select>
      <button @click="toggleTheme" class="bg-gray-800 text-white p-2 rounded">
      Toggle Theme
    </button>
    </div>
    <div class="mb-4 flex">
      <button v-for="(label, index) in listLabels" :key="index" @click="switchList(index)"
              :class="['mr-2 p-2 rounded-t border-t border-l border-r', { 'border-b-2 border-blue-500': currentList === index, 'border-b': currentList !== index }]">
        {{ label }}
      </button>
    </div>
    <table class="w-full border-collapse">
      <thead>
        <tr class="text-gray-500">
          <th class="p-2">Stock</th>
          <th class="p-2">Last</th>
          <th class="p-2">+/-</th>
          <th class="p-2">Buy</th>
          <th class="p-2">Sell</th>
        </tr>
        <tr class="text-gray-500">
          <th class="p-2">Code</th>
          <th class="p-2">Vol</th>
          <th class="p-2">%Chng</th>
          <th class="p-2">Buy Vol</th>
          <th class="p-2">Sell Vol</th>
        </tr>
      </thead>
      <tbody>
        <template v-for="(item, index) in data" :key="index">
          <tr :class="{ 'bg-yellow-300': flashIndices.includes(index) }">
            <td class="border p-2">
              <div style="font-weight: bold;">{{ item.name }}</div>
              <div class="text-gray-500">{{ item.stockcode }}</div>
            </td>
            <td class="border p-2">
              <div>{{ item.last }}</div>
              <div class="text-gray-500">{{ item.volume }}</div>
            </td>
            <td class="border p-2">
              <div :class="{ 'text-red-500': item.change < 0, 'text-green-500': item.change >= 0 }">
                {{ item.change.toFixed(2) }}
              </div>
              <div :class="{ 'text-red-500': item.percentChange < 0, 'text-green-500': item.percentChange >= 0 }">
                {{ item.percentChange.toFixed(2) }}%
              </div>
            </td>
            <td class="border p-2">
              <div>{{ item.buy_price }}</div>
              <div class="text-gray-500">{{ item.buy_volume }}</div>
            </td>
            <td class="border p-2">
              <div>{{ item.sell_price }}</div>
              <div class="text-gray-500">{{ item.sell_volume }}</div>
            </td>
          </tr>
        </template>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  data() {
    return {
      currentMarketId: 0,
      currentList: 0,
      data: [],
      previousData: [],
      flashIndices: [],
      listLabels: ["Top Volume", "Top Gainers", "Top Losers"],
      darkMode: localStorage.getItem("theme") === "dark",
    }
  },
  methods: {
    fetchData() {
      fetch(`https://livefeed3.chartnexus.com/Dummy/quotes?market_id=${this.currentMarketId}&list=${this.currentList}`)
        .then(response => response.json())
        .then(data => {
          this.updateTable(data);
        });
    },
    updateTable(data) {
      this.flashIndices = [];
      data.forEach((item, index) => {
        const prevItem = this.previousData[index];
        const previous = prevItem ? prevItem.last : item.last;
        const change = item.last - previous;
        const percentChange = (change / previous) * 100;
        item.change = change;
        item.percentChange = percentChange;
        if (prevItem && (item.name !== prevItem.name || item.stockcode !== prevItem.stockcode || item.last !== prevItem.last || item.Volume !== prevItem.Volume || item.buy_price !== prevItem.buy_price || item.buy_volume !== prevItem.buy_volume || item.sell_price !== prevItem.sell_price || item.sell_volume !== prevItem.sell_volume)) {
          this.flashIndices.push(index);
        }
      });
      this.previousData = JSON.parse(JSON.stringify(data));
      this.data = data;
    },
    switchList(listIndex) {
      this.currentList = listIndex;
      this.fetchData();
    },
    toggleTheme() {
      this.darkMode = !this.darkMode;
      if (this.darkMode) {
        this.setDarkTheme();
      } else {
        this.setLightTheme();
      }
    },
    setDarkTheme() {
      document.documentElement.classList.add("dark");
      localStorage.setItem("theme", "dark");
    },
    setLightTheme() {
      document.documentElement.classList.remove("dark");
      localStorage.setItem("theme", "light");
    },
  },
  mounted() {
    if (localStorage.getItem("theme") === "dark") {
      this.setDarkTheme();
    } else {
      this.setLightTheme();
    }
    this.fetchData();
    setInterval(this.fetchData, 5000);
  },
}
</script>


<style scoped>
.dark {
  --bg-color: #333;
  --text-color: #fff;
  --highlight-color: #ff0;
  --positive-color: green;
  --negative-color: red;
}

.light {
  --bg-color: #fff;
  --text-color: #333;
  --highlight-color: #ff0;
  --positive-color: green;
  --negative-color: red;
}

.container {
  background-color: var(--bg-color);
  color: var(--text-color);
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
}

select {
  background-color: var(--bg-color);
  color: var(--text-color);
  border: 1px solid #ccc;
  padding: 0.5rem;
  border-radius: 0.25rem;
  width: 150px;
}

.flash {
  background-color: var(--highlight-color);
}

.positive {
  color: var(--positive-color);
}

.negative {
  color: var(--negative-color);
}
</style>
