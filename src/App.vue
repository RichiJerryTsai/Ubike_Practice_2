<script setup>
import { ref, computed, onMounted } from "vue";
import SearchTable from "./components/SearchTable.vue";
import axios from "axios";
import dayjs from "dayjs";

const stations = ref([]);
const dataLoadTime = ref("");
const isLoading = ref(true);

const query = ref("");
const selectedArea = ref("");
const showStationWithBike = ref(false);

const sortField = ref("");
const sortDirection = ref("default");

// 幫忙數字補零，確保格式一致
const pad2 = (value) => String(value).padStart(2, "0");

const formatDate = (date) => {
  return dayjs(date).format("YYYY-MM-DD HH:mm:ss");
};

// 從官方 API 載入 YouBike 站點資料
const loadStations = async () => {
  isLoading.value = true;
  try {
    const response = await axios.get(
      "https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json",
    );

    stations.value =response.data;
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
          station.sarea.toLowerCase().includes(text) ||
          station.ar.toLowerCase().includes(text)
        : true;
      const matchShowStationWithBike = showStationWithBike.value
        ? station.available_rent_bikes > 0
        : true;
      return matchArea && matchText && matchShowStationWithBike;
    })
    .sort((a, b) => b.available_rent_bikes - a.available_rent_bikes);

  if (sortDirection.value === "default" || !sortField.value) {
    return result;
  }
  // 如果是文字用 localeCompare 排序，否則用數字大小排序
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

const changeSort = (field) => {
  if (sortField.value !== field) {
    sortField.value = field;
    sortDirection.value = "asc";
    return;
  }
  switch (sortDirection.value) {
    case "asc":
      sortDirection.value = "desc";
      break;

    case "desc":
      sortDirection.value = "default";
      break;

    default:
      sortDirection.value = "asc";
      break;
  }
};

// 重新整理找到最薪資料
const handleRefresh = () => {
  query.value = "";
  selectedArea.value = "";
  showStationWithBike.value = false;
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
        v-model:showStationWithBike="showStationWithBike"

        :areas="areas"
        :stations="filteredStations"
        :dataLoadTime="dataLoadTime"
        :totalStations="filteredStations.length"

        :sortField="sortField"
        :sortDirection="sortDirection"

        @change-sort="changeSort"
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
