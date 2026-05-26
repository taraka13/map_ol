<template>
  <div class="app-layout">
    <aside class="sidebar sidebar-left">
      <h2>🏙️ Города</h2>
      <CityList
        :cities="cities"
        :selected-id="selectedCity?.id"
        @select="handleSelectCity"
      />
    </aside>

    <main class="main-content">
      <h1>🗺️ Карта городов России</h1>
      <MapComponent
        ref="mapRef"
        :cities="cities"
        :selected-city="selectedCity"
        @marker-click="handleSelectCity"
        @close-popup="selectedCity = null"
      />
    </main>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'
import MapComponent from './components/MapComponent.vue'
import CityList from './components/CityList.vue'
import { cities } from './data/cities.js'

const selectedCity = ref(null)
const mapRef = ref(null)

const handleSelectCity = (city) => {
  // Повторный клик по тому же городу — закрываем popup
  if (selectedCity.value?.id === city.id) {
    selectedCity.value = null
    if (mapRef.value) mapRef.value.clearSelection()
    return
  }
  selectedCity.value = city
  if (mapRef.value) mapRef.value.focusCity(city)
}

watch(selectedCity, (val) => {
  if (!val && mapRef.value) {
    mapRef.value.clearSelection()
  }
})
</script>

<style scoped>
.app-layout {
  display: flex;
  gap: 16px;
  height: calc(100vh - 40px);
}

.sidebar {
  background: #ffffff;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 16px;
  overflow-y: auto;
  box-shadow: 0 2px 8px rgba(0,0,0,0.04);
}

.sidebar-left {
  width: 280px;
  flex-shrink: 0;
}

.main-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-width: 0;
}

.main-content h1 {
  margin-bottom: 12px;
  font-size: 22px;
}
</style>