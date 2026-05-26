<template>
  <div class="map-wrapper">
    <div id="map"></div>

    <!-- Контейнер overlay (управляется OpenLayers, но рендерится Vue) -->
    <div ref="popupRef" class="city-popup" v-show="selectedCity">
      <div v-if="selectedCity" class="popup-content">
        <div class="popup-header">
          <h3>{{ selectedCity.name }}</h3>
          <button class="popup-close" @click.stop="$emit('close-popup')" title="Закрыть">✕</button>
        </div>

        <div class="popup-body">
          <div class="popup-row">
            <span class="label">Страна</span>
            <span class="value">{{ selectedCity.country }}</span>
          </div>
          <div class="popup-row">
            <span class="label">Население</span>
            <span class="value">{{ selectedCity.population }}</span>
          </div>
          <div class="popup-row">
            <span class="label">Основан</span>
            <span class="value">{{ selectedCity.founded }}</span>
          </div>
          <div class="popup-row">
            <span class="label">Координаты</span>
            <span class="value">{{ selectedCity.lat.toFixed(3) }}, {{ selectedCity.lon.toFixed(3) }}</span>
          </div>

          <p class="popup-desc">{{ selectedCity.description }}</p>

          <h4>Достопримечательности</h4>
          <ul class="popup-attractions">
            <li v-for="(a, idx) in selectedCity.attractions" :key="idx">{{ a }}</li>
          </ul>
        </div>

        <div class="popup-arrow"></div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, onUnmounted, ref, watch, nextTick } from 'vue'
import Map from 'ol/Map'
import View from 'ol/View'
import TileLayer from 'ol/layer/Tile'
import VectorLayer from 'ol/layer/Vector'
import VectorSource from 'ol/source/Vector'
import Overlay from 'ol/Overlay'
import OSM from 'ol/source/OSM'
import Feature from 'ol/Feature'
import Point from 'ol/geom/Point'
import { fromLonLat } from 'ol/proj'
import { Circle as CircleStyle, Fill, Stroke, Style, Text } from 'ol/style'
import 'ol/ol.css'

const props = defineProps({
  cities: { type: Array, required: true },
  selectedCity: { type: Object, default: null }
})

const emit = defineEmits(['marker-click', 'close-popup'])

let map = null
let overlay = null
let vectorSource = null
const featuresByCityId = new Map()
const popupRef = ref(null)

const normalStyle = (feature) => new Style({
  image: new CircleStyle({
    radius: 7,
    fill: new Fill({ color: '#e53e3e' }),
    stroke: new Stroke({ color: '#fff', width: 2 })
  }),
  text: new Text({
    text: feature.get('name'),
    offsetY: -16,
    font: '12px Arial',
    fill: new Fill({ color: '#222' }),
    stroke: new Stroke({ color: '#fff', width: 3 })
  })
})

const selectedStyle = (feature) => new Style({
  image: new CircleStyle({
    radius: 10,
    fill: new Fill({ color: '#2b6cb0' }),
    stroke: new Stroke({ color: '#fff', width: 3 })
  }),
  text: new Text({
    text: feature.get('name'),
    offsetY: -20,
    font: 'bold 13px Arial',
    fill: new Fill({ color: '#2b6cb0' }),
    stroke: new Stroke({ color: '#fff', width: 3 })
  })
})

onMounted(() => {
  vectorSource = new VectorSource()

  props.cities.forEach((city) => {
    const feature = new Feature({
      geometry: new Point(fromLonLat([city.lon, city.lat])),
      name: city.name,
      cityId: city.id
    })
    feature.setStyle(normalStyle(feature))
    vectorSource.addFeature(feature)
    featuresByCityId.set(city.id, feature)
  })

  const vectorLayer = new VectorLayer({ source: vectorSource })

  map = new Map({
    target: 'map',
    layers: [new TileLayer({ source: new OSM() }), vectorLayer],
    view: new View({
      center: fromLonLat([55, 55]),
      zoom: 4
    })
  })

  // Создаём overlay, привязанный к нашему popup div
  overlay = new Overlay({
    element: popupRef.value,
    autoPan: {
      animation: { duration: 250 },
      margin: 20
    },
    positioning: 'bottom-center',
    offset: [0, -15],
    stopEvent: true
  })
  map.addOverlay(overlay)

  map.on('click', (evt) => {
    const feature = map.forEachFeatureAtPixel(evt.pixel, (f) => f)
    if (feature && feature.get('cityId')) {
      const city = props.cities.find((c) => c.id === feature.get('cityId'))
      if (city) emit('marker-click', city)
    } else {
      // Клик по пустому месту — закрываем popup
      emit('close-popup')
    }
  })

  map.on('pointermove', (evt) => {
    const hit = map.hasFeatureAtPixel(evt.pixel)
    map.getTargetElement().style.cursor = hit ? 'pointer' : ''
  })
})

onUnmounted(() => {
  if (map) map.setTarget(null)
})

// Когда меняется выбранный город — перемещаем overlay
watch(() => props.selectedCity, async (city) => {
  if (!overlay) return
  if (city) {
    // Даём Vue обновить DOM перед перемещением overlay
    await nextTick()
    overlay.setPosition(fromLonLat([city.lon, city.lat]))
  } else {
    overlay.setPosition(undefined)
  }
}, { immediate: true })

const focusCity = (city) => {
  featuresByCityId.forEach((f, id) => {
    f.setStyle(id === city.id ? selectedStyle(f) : normalStyle(f))
  })
  map.getView().animate({
    center: fromLonLat([city.lon, city.lat]),
    zoom: city.zoom || 10,
    duration: 800
  })
}

const clearSelection = () => {
  featuresByCityId.forEach((f) => f.setStyle(normalStyle(f)))
  if (overlay) overlay.setPosition(undefined)
}

defineExpose({ focusCity, clearSelection })
</script>

<style scoped>
.map-wrapper {
  position: relative;
  flex: 1;
  min-height: 500px;
}

#map {
  width: 100%;
  height: 100%;
  border: 2px solid #ddd;
  border-radius: 8px;
}

/* === Popup стили === */
.city-popup {
  pointer-events: auto;
}

.popup-content {
  position: relative;
  width: 320px;
  max-height: 420px;
  background: #fff;
  border-radius: 10px;
  box-shadow: 0 6px 24px rgba(0,0,0,0.18), 0 2px 6px rgba(0,0,0,0.08);
  overflow: hidden;
  display: flex;
  flex-direction: column;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Arial, sans-serif;
  animation: popupFadeIn 0.25s ease-out;
}

@keyframes popupFadeIn {
  from { opacity: 0; transform: translateY(6px); }
  to   { opacity: 1; transform: translateY(0); }
}

.popup-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 14px;
  background: linear-gradient(135deg, #2b6cb0 0%, #3182ce 100%);
  color: white;
}

.popup-header h3 {
  margin: 0;
  font-size: 17px;
  font-weight: 600;
}

.popup-close {
  background: rgba(255,255,255,0.2);
  border: none;
  color: white;
  width: 26px;
  height: 26px;
  border-radius: 50%;
  cursor: pointer;
  font-size: 14px;
  line-height: 1;
  transition: background 0.15s;
}

.popup-close:hover {
  background: rgba(255,255,255,0.35);
}

.popup-body {
  padding: 12px 14px;
  overflow-y: auto;
  flex: 1;
  font-size: 13px;
}

.popup-row {
  display: flex;
  justify-content: space-between;
  padding: 5px 0;
  border-bottom: 1px dashed #eee;
}

.popup-row .label {
  color: #666;
}

.popup-row .value {
  font-weight: 600;
  color: #222;
}

.popup-desc {
  margin: 10px 0 6px;
  line-height: 1.45;
  color: #444;
}

.popup-body h4 {
  margin: 10px 0 4px;
  font-size: 13px;
  color: #2b6cb0;
}

.popup-attractions {
  padding-left: 16px;
  margin: 0;
  line-height: 1.55;
  color: #444;
}

/* Стрелка popup снизу */
.popup-arrow {
  position: absolute;
  bottom: -9px;
  left: 50%;
  transform: translateX(-50%) rotate(45deg);
  width: 16px;
  height: 16px;
  background: #fff;
  box-shadow: 2px 2px 4px rgba(0,0,0,0.08);
}
</style>