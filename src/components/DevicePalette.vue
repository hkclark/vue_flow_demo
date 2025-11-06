<script setup>
const props = defineProps({
  devicePalette: Array,
  shapePalette: Array,
  connectionTypes: Array,
  selectedConnectionType: String,
  showCiscoDevices: Boolean,
  showBasicShapes: Boolean
})

const emit = defineEmits([
  'update:selectedConnectionType',
  'update:showCiscoDevices',
  'update:showBasicShapes'
])

const onDragStart = (event, item) => {
  event.dataTransfer.effectAllowed = 'move'
  event.dataTransfer.setData('application/json', JSON.stringify(item))
}

const toggleSection = (section) => {
  if (section === 'cisco') {
    emit('update:showCiscoDevices', !props.showCiscoDevices)
  } else if (section === 'basic') {
    emit('update:showBasicShapes', !props.showBasicShapes)
  }
}
</script>

<template>
  <div class="palette">
    <div class="palette-header">
      <h3>Shapes</h3>
    </div>

    <div class="palette-section">
      <button
        class="section-header"
        :class="{ active: showBasicShapes }"
        @click="toggleSection('basic')"
      >
        <span class="chevron">{{ showBasicShapes ? '▼' : '▶' }}</span>
        <span>Basic</span>
      </button>
      <div v-show="showBasicShapes" class="palette-items">
        <div
          v-for="item in shapePalette"
          :key="item.type"
          class="palette-item"
          draggable="true"
          @dragstart="onDragStart($event, item)"
        >
          <div class="item-icon" v-html="wrapSvg(item.svg)"></div>
          <span class="item-label">{{ item.label }}</span>
        </div>
      </div>
    </div>

    <div class="palette-section">
      <button
        class="section-header"
        :class="{ active: showCiscoDevices }"
        @click="toggleSection('cisco')"
      >
        <span class="chevron">{{ showCiscoDevices ? '▼' : '▶' }}</span>
        <span>Cisco</span>
      </button>
      <div v-show="showCiscoDevices" class="palette-items">
        <div
          v-for="item in devicePalette"
          :key="item.type"
          class="palette-item"
          draggable="true"
          @dragstart="onDragStart($event, item)"
        >
          <div class="item-icon" v-html="item.svg"></div>
          <span class="item-label">{{ item.label }}</span>
        </div>
      </div>
    </div>

    <div class="connection-section">
      <h4>Connection Type</h4>
      <div class="connection-types">
        <button
          v-for="conn in connectionTypes"
          :key="conn.type"
          @click="emit('update:selectedConnectionType', conn.type)"
          class="connection-btn"
          :class="{ active: selectedConnectionType === conn.type }"
        >
          <span class="conn-label">{{ conn.label }}</span>
          <svg width="48" height="4" class="conn-preview">
            <line
              x1="0"
              y1="2"
              x2="48"
              y2="2"
              :stroke="conn.color"
              stroke-width="2"
              :stroke-dasharray="conn.style.strokeDasharray"
            />
          </svg>
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  methods: {
    wrapSvg(svgContent) {
      return `<svg width="50" height="50" viewBox="0 0 50 50">${svgContent}</svg>`
    }
  }
}
</script>

<style scoped>
.palette {
  width: 280px;
  background: white;
  border-right: 1px solid #dee2e6;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
}

.palette-header {
  padding: 16px 20px;
  border-bottom: 1px solid #dee2e6;
}

.palette-header h3 {
  font-size: 16px;
  font-weight: 600;
  color: #212529;
  margin: 0;
}

.palette-section {
  border-bottom: 1px solid #f1f3f5;
}

.section-header {
  width: 100%;
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 12px 20px;
  background: white;
  border: none;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  color: #495057;
  text-align: left;
  transition: background 0.2s;
}

.section-header:hover {
  background: #f8f9fa;
}

.section-header.active {
  color: #0d6efd;
}

.chevron {
  font-size: 10px;
  color: #6c757d;
  width: 12px;
}

.palette-items {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
  padding: 16px;
  background: #fafafa;
}

.palette-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  padding: 12px 8px;
  background: white;
  border: 1px solid #dee2e6;
  border-radius: 4px;
  cursor: move;
  transition: all 0.2s;
}

.palette-item:hover {
  border-color: #0d6efd;
  box-shadow: 0 2px 4px rgba(13, 110, 253, 0.15);
  transform: translateY(-1px);
}

.item-icon {
  width: 50px;
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.item-icon :deep(svg) {
  max-width: 100%;
  max-height: 100%;
}

.item-label {
  font-size: 11px;
  color: #495057;
  text-align: center;
  font-weight: 500;
}

.connection-section {
  padding: 16px 20px;
  border-top: 1px solid #dee2e6;
  margin-top: auto;
}

.connection-section h4 {
  font-size: 14px;
  font-weight: 600;
  color: #212529;
  margin: 0 0 12px 0;
}

.connection-types {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.connection-btn {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #dee2e6;
  border-radius: 4px;
  background: white;
  cursor: pointer;
  transition: all 0.2s;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.connection-btn:hover {
  border-color: #0d6efd;
  background: #f8f9fa;
}

.connection-btn.active {
  background: #e7f1ff;
  border-color: #0d6efd;
}

.conn-label {
  font-size: 13px;
  font-weight: 500;
  color: #495057;
}

.conn-preview {
  flex-shrink: 0;
}
</style>

