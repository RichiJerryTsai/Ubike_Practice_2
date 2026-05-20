<script setup>
import { ref, computed, onMounted } from "vue";
import SearchTable from "./components/SearchTable.vue";

const stations = ref([]);
const dataLoadTime = ref("");
const isLoading = ref(true);

const query = ref("");
const selectedArea = ref("");
const showAll = ref(false);

const sortField = ref("");
const sortDirection = ref("default");

// 幫忙數字補零，確保格式一致
const pad2 = (value) => String(value).padStart(2, "0");

//格式化日期時間，月份要特別注意 + 1
const formatDate = (date) =>
  `${date.getFullYear()}-${pad2(date.getMonth() + 1)}-${pad2(date.getDate())} ${pad2(date.getHours())}:${pad2(date.getMinutes())}:${pad2(date.getSeconds())}`;

// 從官方 API 載入 YouBike 站點資料
const loadStations = async () => {
  isLoading.value = true;
  try {
    const response = await fetch(
      "https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json",
    );
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    stations.value = await response.json();
    dataLoadTime.value = formatDate(new Date());
  } catch (error) {
    console.error("Failed to Load YouBike data:", error);
    stations.value = [];
    dataLoadTime.value = "Failed to Load Data";
  } finally {
    isLoading.value = false;
  }
};

// 找到唯一的行政區名稱並且排序
const areas = computed(() => {
  return [...new Set(stations.value.map((s) => s.sarea))].sort(
    (a, b) => a.length - b.length,
  );
});

// 預設為依照可租借車輛數量排序，根據使用者輸入的文字來查找站點
const filteredStations = computed(() => {
  const text = query.value.trim().toLowerCase();
  let result = stations.value
    .filter((station) => {
      const matchArea = selectedArea.value
        ? station.sarea === selectedArea.value
        : true;
      const matchText = text
        ? station.sna.toLowerCase().includes(text) ||
          station.sarea.toLowerCase().includes(text)
        : true;
      const matchShowAll = showAll.value
        ? station.available_rent_bikes > 0
        : true;
      return matchArea && matchText && matchShowAll;
    })
    .sort((a, b) => b.available_rent_bikes - a.available_rent_bikes);

  if (sortDirection.value === "default" || !sortField.value) {
    return result;
  }

  result.sort((a, b) => {
    const field = sortField.value;
    if (field === "sna" || field === "sarea" || field === "ar") {
      return sortDirection.value === "asc"
        ? a[field].localeCompare(b[field])
        : b[field].localeCompare(a[field]);
    }
    return sortDirection.value === "asc"
      ? a[field] - b[field]
      : b[field] - a[field];
  });

  return result;
});

// 重新整理找到最薪資料
const handleRefresh = () => {
  query.value = "";
  selectedArea.value = "";
  showAll.value = false;
  loadStations();
};

onMounted(() => {
  loadStations();
});
</script>

<template>
  <main class="app-container">
    <section class="results">
      <SearchTable
        v-model:query="query"
        v-model:selectedArea="selectedArea"
        v-model:showAll="showAll"
        :areas="areas"
        :stations="filteredStations"
        :dataLoadTime="dataLoadTime"
        :totalStations="filteredStations.length"
        @refresh="handleRefresh"
      />
    </section>
  </main>
</template>

<style scoped>
.app-container {
  width: 100%;
  margin: 0;
  padding: 16px;
  box-sizing: border-box;
}

.results {
  margin-top: 28px;
}
</style>
