<script setup>
const query = defineModel("query");
const selectedArea = defineModel("selectedArea");
const showAll = defineModel("showAll");

defineProps({
  areas: {
    type: Array,
    required: true,
  },
});

const emit = defineEmits(["refresh"]);
</script>

<template>
  <div class="search-panel">
    <label>
      <span>站名/地點查詢：</span>
      <input type="search" v-model="query" />
    </label>
    <label>
      <span>場站區域：</span>
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
    <button type="button" @click="emit('refresh')">
      <img src="/Pics/reload.svg" alt="Reload" class="reload-icon" />
      最新資料
    </button>
  </div>
</template>

<style scoped>
.search-panel {
  display: grid;
  gap: 10px;
  grid-template-columns: repeat(3, 1fr) 100px;
  align-items: end;
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
  gap: 8px;
}

.search-panel input,
.search-panel select {
  width: 400px;
  height: 50px;
  border: 1px solid var(--border);
  padding: 0 16px;
  background: #ffffff;
  color: var(--text);
  font: inherit;
  align-items: center;
}

.search-panel input:focus,
.search-panel select:focus {
  outline: 2px solid rgba(37, 99, 235, 0.25);
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
</style>
