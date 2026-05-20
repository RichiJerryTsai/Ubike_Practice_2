<script setup>
import { computed, ref } from "vue";
const query = defineModel("query");
const selectedArea = defineModel("selectedArea");
const showAll = defineModel("showAll");

defineProps({
  areas: {
    type: Array,
    required: true,
  },
  stations: {
    type: Array,
    required: true,
  },
  dataLoadTime: {
    type: String,
    required: true,
  },
  totalStations: {
    type: Number,
    required: true,
  },
});

const emit = defineEmits(["refresh"]);
const formatName = (sna) => sna.replace("YouBike2.0_", "");
const formatCoord = (lat, lng) => `(${lat.toFixed(6)}, ${lng.toFixed(6)})`;
const openGoogleMaps = (lat, lng) => {
  window.open(`https://www.google.com/maps?q=${lat},${lng}`, "_blank");
};

const changeSort =(field) =>{
    if(field === "sarea" || field === "sna" || field === "ar"){
      filteredStations.value.sort((a, b) => a[field].localeCompare(b[field]));
    }
    else if(field === "available_rent_bikes" || field === "available_return_bikes"){
      filteredStations.value.sort((a, b) => b[field] - a[field]);
    };
};
</script>

<template>
  <div class="search-panel">
    <label>
      <span>站名地點查詢： </span>
      <input type="search" v-model="query" />
    </label>

    <label>
      <span>場站區域： </span>
      <select v-model="selectedArea">
        <option value="">不拘</option>
        <option v-for="area in areas" :key="area" :value="area">
          {{ area }}
        </option>
      </select>
    </label>

    <label class="checkbox-field">
      <input type="checkbox" v-model="showAll" />
      <span>只顯示有車輛的站點</span>
    </label>
    <button class="secondary" type="button" @click="$emit('refresh')">
      <img src="/Pics/reload.svg" alt="Reload" class="reload-icon" />
      最新資料
    </button>
  </div>
  <div class="result-panel">
    <div class="result-table-container">
      <div class="result-table-wrapper">
        <table class="result-table">
          <thead>
            <tr>
              <th>項次</th>
              <th>
                 場站區域
                 <button class="sort-btn" @click="changeSort('sarea')">
                   <img src="/Pics/sort.svg" alt="排序" class="sort-icon" />
                 </button>
               </th> 
              <th>
                站名
                <button class="sort-btn" @click="changeSort('sna')">
                  <img src="/Pics/sort.svg" alt="排序" class="sort-icon" />
                </button>
              </th>
              <th>
                地點
                <button class="sort-btn" @click="changeSort('ar')">
                  <img src="/Pics/sort.svg" alt="排序" class="sort-icon" />
                </button>
              </th>
              <th>
                坐標位置
                <button class="sort-btn" @click="">
                  <img src="/Pics/sort.svg" alt="排序" class="sort-icon" />
                </button>
              </th>
              <th>
                目前車輛數
                <button class="sort-btn" @click="changeSort('available_rent_bikes')">
                  <img src="/Pics/sort.svg" alt="排序" class="sort-icon" />
                </button>
              </th>
              <th>
                目前空位數
                <button class="sort-btn" @click="changeSort('available_return_bikes')">
                  <img src="/Pics/sort.svg" alt="排序" class="sort-icon" />
                </button>
              </th>
            </tr>
          </thead>
          <tbody>
            <!--顯示前50筆資料，以及資料更新時間-->
            <tr
              v-for="(station, index) in stations.slice(0, 50)"
              :key="station.sno"
              :title="`資料時間：${dataLoadTime}`"
            >
              <td>{{ index + 1 }}</td>
              <td>{{ station.sarea }}</td>
              <td>{{ formatName(station.sna) }}</td>
              <td>{{ station.ar }}</td>
              <td class="coord-cell">
                <button
                  class="location-btn"
                  @click="openGoogleMaps(station.latitude, station.longitude)"
                >
                  <img src="/Pics/location.jpg" alt="location" />
                </button>
                {{ formatCoord(station.latitude, station.longitude) }}
              </td>
              <td>{{ station.available_rent_bikes }}</td>
              <td>{{ station.available_return_bikes }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
  <div class="table-footer">
    <div>資料筆數：{{ totalStations }}</div>
    <div>資料更新時間：{{ dataLoadTime }}</div>
  </div>
</template>

<style scoped>
/* Search Panel */
.search-panel {
  display: grid;
  gap: 30px;
  grid-template-columns: repeat(3, 1fr) 100px;
  align-items: center;
  width: 100%;
}

.search-panel label span {
  white-space: nowrap;
  flex-shrink: 0;
}

.search-panel label {
  display: flex;
  gap: 8px;
  text-align: left;
  color: var(--text);
  margin: 0;
  align-items: center;
}

.search-panel .checkbox-field {
  display: inline-flex;
  align-items: center;
  gap: 16px;
}

.search-panel input,
.search-panel select {
  width: 100%;
  height: 50px;
  border: 1px solid var(--border);
  padding: 0 16px;
  font-size: 16px;
}

.search-panel input:focus,
.search-panel select:focus {
  outline: 2px solid #77a2ff40;
}

.checkbox-field {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 16px;
  color: var(--text);
}

.checkbox-field input[type="checkbox"] {
  width: 18px;
  height: 18px;
  margin: 0;
}

.reload-icon {
  width: 12px;
  height: 12px;
  margin-right: 4px;
  vertical-align: middle;
}

/* Result Table */
.result-table-container {
  border: 1px solid var(--border);
  overflow: hidden;
  background: var(--bg);
  margin: 16px 0;
}
/* 固定表格表頭 */
.result-table thead th {
  position: sticky;
  top: 0;
  z-index: 2;
}
/*限制表格高度*/
.result-table-wrapper {
  max-height: 700px;
  overflow-y: auto;
  overflow-x: hidden;
}

.result-table {
  width: 100%;
  border-collapse: collapse;
  background: #ffffff;
  border-left: 1px solid var(--border);
  border-right: 1px solid var(--border);
}

.result-table th,
.result-table td {
  padding: 14px 16px;
  border-bottom: 1px solid #e5e7eb;
  text-align: center;
}

.result-table th {
  background: #239bb0;
  color: var(--bg);
  font-weight: 700;
  text-align: center;
  border-left: 1px solid var(--border);
  border-right: 1px solid var(--border);
}

.result-table tbody tr:nth-child(odd) {
  background: #f2f2f2;
}

.result-table tbody tr:last-child td {
  border-bottom: none;
}

.sort-btn {
  background: transparent;
  border: none;
  padding: 0;
  margin-left: 4px;
  cursor: pointer;
}

.sort-icon {
  width: 16px;
  height: 16px;
  display: block;
}

.coord-cell {
  display: flex;
  align-items: center;
  gap: 8px;
}

.sort-icon {
  width: 16px;
  height: 16px;
  margin-left: 4px;
  opacity: 0.6;
  align-items: center;
}

.location-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  padding: 0;
  border: none;
  background: none;
  flex-shrink: 0;
}

.location-btn img {
  width: 20px;
  height: 20px;
  object-fit: contain;
}

.location-btn:hover {
  transform: scale(1.2);
  transition: transform 0.15s ease;
}

.table-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  min-height: 50px;
  padding: 0 16px;
  background: #ffffff;
  border: 1px solid var(--border);
  border-top: 1px solid var(--border);
  color: var(--muted);
}
</style>
