<script setup>
import { ref, watch, onMounted, computed } from 'vue'

const props = defineProps({
  edge: Object
})

const emit = defineEmits(['close', 'update', 'toFront', 'toBack'])

const modalRef = ref(null)
const isDragging = ref(false)
const position = ref({ x: 150, y: 150 })
const dragStart = ref({ x: 0, y: 0 })

const formData = ref({
  id: '',
  centerLabel: '',
  sourceLabel: '',
  targetLabel: '',
  strokeWidth: 2,
  strokeColor: '#005073',
  lineStyle: 'solid',
  labelFontSize: 12,
  labelColor: '#005073',
  sourceMarker: 'none',
  targetMarker: 'none'
})

watch(() => props.edge, (newEdge) => {
  if (newEdge) {
    formData.value = {
      id: newEdge.id,
      centerLabel: newEdge.data?.centerLabel || newEdge.label || '',
      sourceLabel: newEdge.data?.sourceLabel || '',
      targetLabel: newEdge.data?.targetLabel || '',
      strokeWidth: newEdge.data?.strokeWidth || 2,
      strokeColor: newEdge.data?.strokeColor || newEdge.style?.stroke || '#005073',
      lineStyle: newEdge.data?.lineStyle ? 
        (newEdge.data.lineStyle === '' ? 'solid' : 
         newEdge.data.lineStyle === '8,4' ? 'dashed' : 'dotted') : 'solid',
      labelFontSize: newEdge.data?.labelFontSize || 12,
      labelColor: newEdge.data?.labelColor || '#005073',
      sourceMarker: newEdge.data?.sourceMarker || (newEdge.markerStart ? newEdge.markerStart.type : 'none'),
      targetMarker: newEdge.data?.targetMarker || (newEdge.markerEnd ? newEdge.markerEnd.type : 'none')
    }
    
    // Position modal away from the edge, towards bottom-right
    const rect = document.querySelector('.vue-flow-canvas')?.getBoundingClientRect()
    if (rect) {
      // Default to bottom-right area
      position.value = {
        x: Math.max(50, rect.width - 680),
        y: Math.max(50, Math.min(200, rect.height - 450))
      }
    }
  }
}, { immediate: true })

const handleSubmit = () => {
  emit('update', formData.value)
}

const startDrag = (e) => {
  if (e.target.closest('.modal-body') || e.target.closest('button:not(.drag-handle)')) return
  
  isDragging.value = true
  dragStart.value = {
    x: e.clientX - position.value.x,
    y: e.clientY - position.value.y
  }
}

const onDrag = (e) => {
  if (!isDragging.value) return
  position.value = {
    x: e.clientX - dragStart.value.x,
    y: e.clientY - dragStart.value.y
  }
}

const stopDrag = () => {
  isDragging.value = false
}

onMounted(() => {
  document.addEventListener('mousemove', onDrag)
  document.addEventListener('mouseup', stopDrag)
})
</script>

<template>
  <div 
    ref="modalRef"
    class="draggable-modal compact" 
    :style="{ left: position.x + 'px', top: position.y + 'px' }"
  >
    <div class="modal-header" @mousedown="startDrag">
      <svg width="14" height="14" viewBox="0 0 16 16" class="drag-icon">
        <circle cx="4" cy="4" r="1.5" fill="#999"/>
        <circle cx="8" cy="4" r="1.5" fill="#999"/>
        <circle cx="12" cy="4" r="1.5" fill="#999"/>
        <circle cx="4" cy="8" r="1.5" fill="#999"/>
        <circle cx="8" cy="8" r="1.5" fill="#999"/>
        <circle cx="12" cy="8" r="1.5" fill="#999"/>
        <circle cx="4" cy="12" r="1.5" fill="#999"/>
        <circle cx="8" cy="12" r="1.5" fill="#999"/>
        <circle cx="12" cy="12" r="1.5" fill="#999"/>
      </svg>
      <h3>Connection Details</h3>
      <button @click="emit('close')" class="close-btn">×</button>
    </div>
    
    <div class="modal-body">
      <div class="form-group">
        <label>Center Label</label>
        <input v-model="formData.centerLabel" type="text" class="form-control" placeholder="Center label (optional)">
      </div>

      <div class="form-row-2">
        <div class="form-group">
          <label>Source Label</label>
          <input v-model="formData.sourceLabel" type="text" class="form-control" placeholder="Source">
        </div>
        <div class="form-group">
          <label>Target Label</label>
          <input v-model="formData.targetLabel" type="text" class="form-control" placeholder="Target">
        </div>
      </div>

      <div class="section-title">Line Style</div>

      <div class="form-row-3">
        <div class="form-group">
          <label>Width</label>
          <select v-model.number="formData.strokeWidth" class="form-control">
            <option :value="1">1px</option>
            <option :value="2">2px</option>
            <option :value="3">3px</option>
            <option :value="4">4px</option>
            <option :value="5">5px</option>
          </select>
        </div>
        <div class="form-group">
          <label>Color</label>
          <input v-model="formData.strokeColor" type="color" class="form-control color-input">
        </div>
        <div class="form-group">
          <label>Style</label>
          <select v-model="formData.lineStyle" class="form-control">
            <option value="solid">Solid</option>
            <option value="dashed">Dashed</option>
            <option value="dotted">Dotted</option>
          </select>
        </div>
      </div>

      <div class="form-row-2">
        <div class="form-group">
          <label>Source End</label>
          <select v-model="formData.sourceMarker" class="form-control">
            <option value="none">None</option>
            <option value="arrow">Arrow</option>
            <option value="arrowclosed">Closed Arrow</option>
          </select>
        </div>
        <div class="form-group">
          <label>Target End</label>
          <select v-model="formData.targetMarker" class="form-control">
            <option value="none">None</option>
            <option value="arrow">Arrow</option>
            <option value="arrowclosed">Closed Arrow</option>
          </select>
        </div>
      </div>

      <div class="section-title">Label Style</div>

      <div class="form-row-2">
        <div class="form-group">
          <label>Size</label>
          <input v-model.number="formData.labelFontSize" type="number" min="8" max="32" class="form-control">
        </div>
        <div class="form-group">
          <label>Color</label>
          <input v-model="formData.labelColor" type="color" class="form-control color-input">
        </div>
      </div>

      <div class="section-title">Actions</div>
      <div class="action-row">
        <button @click="emit('toFront', formData.id)" class="action-btn">⬆️ To Front</button>
        <button @click="emit('toBack', formData.id)" class="action-btn">⬇️ To Back</button>
      </div>
    </div>

    <div class="modal-footer">
      <button @click="handleSubmit" class="btn btn-apply">✓ Apply Changes</button>
    </div>
  </div>
</template>

<style scoped>
.draggable-modal {
  position: fixed;
  background: white;
  border-radius: 6px;
  width: 620px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.25);
  z-index: 1000;
  display: flex;
  flex-direction: column;
}

.draggable-modal.compact .modal-body {
  padding: 12px;
}

.draggable-modal.compact .form-group {
  margin-bottom: 8px;
}

.modal-header {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 8px 12px;
  border-bottom: 1px solid #e5e7eb;
  cursor: move;
  user-select: none;
  background: #f9fafb;
  border-radius: 6px 6px 0 0;
}

.drag-icon {
  flex-shrink: 0;
}

.modal-header h3 {
  margin: 0;
  font-size: 14px;
  font-weight: 600;
  color: #111827;
  flex: 1;
}

.close-btn {
  background: none;
  border: none;
  font-size: 20px;
  color: #6b7280;
  cursor: pointer;
  padding: 0;
  width: 20px;
  height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 3px;
  flex-shrink: 0;
}

.close-btn:hover {
  background: #e5e7eb;
}

.modal-body {
  max-height: 65vh;
  overflow-y: auto;
}

.section-title {
  font-size: 12px;
  font-weight: 600;
  color: #374151;
  margin: 12px 0 8px 0;
  padding-bottom: 3px;
  border-bottom: 1px solid #e5e7eb;
}

.form-group {
  margin-bottom: 10px;
}

.form-group label {
  display: block;
  font-size: 11px;
  font-weight: 500;
  color: #4b5563;
  margin-bottom: 3px;
}

.form-control {
  width: 100%;
  padding: 5px 8px;
  border: 1px solid #d1d5db;
  border-radius: 3px;
  font-size: 12px;
  font-family: inherit;
}

.form-control:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.1);
}

.color-input {
  height: 30px;
  padding: 2px;
}

.form-row-2 {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}

.form-row-3 {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  gap: 10px;
}

.action-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 8px;
}

.action-btn {
  padding: 6px 10px;
  border: 1px solid #d1d5db;
  border-radius: 3px;
  background: white;
  cursor: pointer;
  font-size: 11px;
  text-align: left;
  transition: all 0.2s;
}

.action-btn:hover {
  background: #f3f4f6;
  border-color: #9ca3af;
}

.modal-footer {
  padding: 8px 12px;
  border-top: 1px solid #e5e7eb;
  display: flex;
  justify-content: flex-end;
}

.btn-apply {
  background: #2563eb;
  color: white;
  border: none;
  padding: 6px 20px;
  border-radius: 4px;
  font-size: 12px;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.2s;
}

.btn-apply:hover {
  background: #1d4ed8;
}
</style>

