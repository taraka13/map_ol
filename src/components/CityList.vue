<template>
  <div class="city-list">
    <div
      v-for="city in cities"
      :key="city.id"
      class="city-item"
      :class="{ active: selectedId === city.id }"
      @click="$emit('select', city)"
    >
      <div class="city-name">{{ city.name }}</div>
      <div class="city-meta">{{ city.country }} • {{ city.population }}</div>
    </div>
  </div>
</template>

<script setup>
defineProps({
  cities: { type: Array, required: true },
  selectedId: { type: Number, default: null }
})

defineEmits(['select'])
</script>

<style scoped>
.city-list {
  display: flex;
  flex-direction: column;
  gap: 6px;
  margin-top: 8px;
}

.city-item {
  padding: 10px 12px;
  border-radius: 6px;
  cursor: pointer;
  border: 1px solid transparent;
  transition: all 0.15s ease;
  background: #f7f7f7;
}

.city-item:hover {
  background: #eef5ff;
  border-color: #b8d4ff;
}

.city-item.active {
  background: #2b6cb0;
  color: white;
  border-color: #2b6cb0;
}

.city-item.active .city-meta {
  color: #d8e6f5;
}

.city-name {
  font-weight: 600;
  font-size: 15px;
  margin-bottom: 3px;
}

.city-meta {
  font-size: 12px;
  color: #777;
}

h2 {
  margin: 0 0 8px 0;
  font-size: 18px;
}
</style>