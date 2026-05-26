<template>
  <div class="city-info">
    <div class="info-header">
      <h2>{{ city.name }}</h2>
      <button class="close-btn" @click="$emit('close')" title="Закрыть">✕</button>
    </div>

    <div class="info-body">
      <div class="info-row">
        <span class="label">Страна</span>
        <span class="value">{{ city.country }}</span>
      </div>
      <div class="info-row">
        <span class="label">Население</span>
        <span class="value">{{ city.population }}</span>
      </div>
      <div class="info-row">
        <span class="label">Основан</span>
        <span class="value">{{ city.founded }}</span>
      </div>
      <div class="info-row">
        <span class="label">Координаты</span>
        <span class="value">{{ city.lat.toFixed(4) }}, {{ city.lon.toFixed(4) }}</span>
      </div>

      <h3>Описание</h3>
      <p class="description">{{ city.description }}</p>

      <h3>Достопримечательности</h3>
      <ul class="attractions">
        <li v-for="(a, idx) in city.attractions" :key="idx">{{ a }}</li>
      </ul>
    </div>
  </div>
</template>

<script setup>
defineProps({
  city: { type: Object, required: true }
})

defineEmits(['close'])
</script>

<style scoped>
.city-info {
  height: 100%;
  display: flex;
  flex-direction: column;
}

.info-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid #e0e0e0;
  padding-bottom: 10px;
  margin-bottom: 14px;
}

.info-header h2 {
  margin: 0;
  font-size: 20px;
  color: #2b6cb0;
}

.close-btn {
  background: transparent;
  border: none;
  font-size: 18px;
  cursor: pointer;
  color: #666;
  width: 28px;
  height: 28px;
  border-radius: 50%;
  padding: 0;
}

.close-btn:hover {
  background: #eee;
  color: #000;
}

.info-body {
  flex: 1;
}

.info-row {
  display: flex;
  justify-content: space-between;
  padding: 6px 0;
  border-bottom: 1px dashed #eee;
  font-size: 14px;
}

.label {
  color: #666;
}

.value {
  font-weight: 600;
  color: #222;
}

h3 {
  margin: 16px 0 6px;
  font-size: 15px;
  color: #333;
}

.description {
  font-size: 14px;
  line-height: 1.5;
  color: #444;
}

.attractions {
  padding-left: 18px;
  font-size: 14px;
  color: #444;
  line-height: 1.7;
}
</style>