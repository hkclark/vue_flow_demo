<script setup>
import { ref, watch, onMounted, onUnmounted, computed } from 'vue'

const props = defineProps({
  node: Object
})

const emit = defineEmits(['close', 'update', 'duplicate', 'toFront', 'toBack', 'delete'])

const modalRef = ref(null)
const isDragging = ref(false)
const position = ref({ x: 100, y: 100 })
const dragStart = ref({ x: 0, y: 0 })

const formData = ref({
  id: '',
  label: '',
  labelPosition: 'bottom',
  labelColor: '#495057',
  labelFontSize: 11,
  labelBold: false,
  labelItalic: false,
  labelUnderline: false,
  fillColor: '#ffffff',
  borderColor: '#005073',
  borderWidth: 2,
  width: 70,
  height: 70,
  notes: '',
  url: '',
  tags: [],
  locked: false
})

const newTag = ref('')

const addTag = () => {
  if (newTag.value.trim() && !formData.value.tags.includes(newTag.value.trim())) {
    formData.value.tags.push(newTag.value.trim())
    newTag.value = ''
  }
}

const removeTag = (index) => {
  formData.value.tags.splice(index, 1)
}

const isIcon = computed(() => formData.value.isIcon)

watch(() => props.node, (newNode) => {
  if (newNode) {
    formData.value = {
      id: newNode.id,
      label: newNode.data.label,
      labelPosition: newNode.data.labelPosition || 'bottom',
      labelColor: newNode.data.labelColor || '#495057',
      labelFontSize: newNode.data.labelFontSize || 11,
      labelBold: newNode.data.labelBold || false,
      labelItalic: newNode.data.labelItalic || false,
      labelUnderline: newNode.data.labelUnderline || false,
      fillColor: newNode.data.fillColor || '#ffffff',
      borderColor: newNode.data.borderColor || '#005073',
      borderWidth: newNode.data.borderWidth || 2,
      width: newNode.data.width || 70,
      height: newNode.data.height || 70,
      svg: newNode.data.svg,
      deviceType: newNode.data.deviceType,
      isIcon: newNode.data.isIcon,
      notes: newNode.data.notes || '',
      url: newNode.data.url || '',
      tags: newNode.data.tags || [],
      locked: newNode.data.locked || false
    }

    // Position modal in a fixed position - top center of screen
    // This way it never covers any nodes
    position.value = {
      x: window.innerWidth / 2 - 290,
      y: 100
    }
  }
}, { immediate: true })

const handleSubmit = () => {
  emit('update', formData.value)
}

const startDrag = (e) => {
  if (e.target.closest('.modal-body') || e.target.closest('button')) return

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

const handleKeydown = (e) => {
  if (e.key === 'Escape') {
    emit('close')
  }
}

onMounted(() => {
  document.addEventListener('mousemove', onDrag)
  document.addEventListener('mouseup', stopDrag)
  document.addEventListener('keydown', handleKeydown)
})

onUnmounted(() => {
  document.removeEventListener('mousemove', onDrag)
  document.removeEventListener('mouseup', stopDrag)
  document.removeEventListener('keydown', handleKeydown)
})
</script>

<template>
  <div
    ref="modalRef"
    class="draggable-modal"
    :style="{ left: position.x + 'px', top: position.y + 'px' }"
  >
    <div class="modal-header" @mousedown="startDrag">
      <svg width="16" height="16" viewBox="0 0 16 16" class="drag-icon">
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
      <h3>Edit Shape</h3>
      <button @click="emit('close')" class="close-btn">×</button>
    </div>

    <div class="modal-body">
      <div class="two-column">
        <div class="left-column">
          <div class="section-title">Label Customization</div>

          <div class="form-group">
            <label>Label Text</label>
            <input v-model="formData.label" type="text" class="form-control">
          </div>

          <div class="form-group">
            <label>Label Position</label>
            <select v-model="formData.labelPosition" class="form-control">
              <option value="top">Above</option>
              <option value="bottom">Below</option>
              <option value="left">Left</option>
              <option value="right">Right</option>
              <option value="center">Center</option>
            </select>
          </div>

          <div class="form-group">
            <label>Label Color</label>
            <input v-model="formData.labelColor" type="color" class="form-control color-input">
          </div>

          <div class="form-group">
            <label>Font Size</label>
            <input v-model.number="formData.labelFontSize" type="number" min="8" max="32" class="form-control">
          </div>

          <div class="form-group">
            <label>Label Style</label>
            <div class="button-group">
              <button
                @click="formData.labelBold = !formData.labelBold"
                class="style-btn"
                :class="{ active: formData.labelBold }"
              >
                <strong>B</strong>
              </button>
              <button
                @click="formData.labelItalic = !formData.labelItalic"
                class="style-btn"
                :class="{ active: formData.labelItalic }"
              >
                <em>I</em>
              </button>
              <button
                @click="formData.labelUnderline = !formData.labelUnderline"
                class="style-btn"
                :class="{ active: formData.labelUnderline }"
              >
                <u>U</u>
              </button>
            </div>
          </div>
        </div>

        <div class="right-column">
          <div class="section-title">Actions</div>

          <button @click="emit('duplicate', formData.id)" class="action-btn">
            📋 Duplicate
          </button>
          <button @click="emit('toFront', formData.id)" class="action-btn">
            ⬆️ To Front
          </button>
          <button @click="emit('toBack', formData.id)" class="action-btn">
            ⬇️ To Back
          </button>

          <div class="section-title" style="margin-top: 16px;">Shape Styling</div>

          <div class="form-group">
            <label>Fill Color</label>
            <div class="color-with-checkbox">
              <input
                v-model="formData.fillColor"
                type="color"
                class="form-control color-input"
                :disabled="formData.fillColor === 'none'"
              >
              <label class="checkbox-inline">
                <input
                  type="checkbox"
                  :checked="formData.fillColor === 'none'"
                  @change="formData.fillColor = $event.target.checked ? 'none' : '#ffffff'"
                >
                <span>None</span>
              </label>
            </div>
          </div>

          <div class="form-group">
            <label>Border Color</label>
            <input v-model="formData.borderColor" type="color" class="form-control color-input">
          </div>

          <div class="form-group">
            <label>Border Width</label>
            <select v-model.number="formData.borderWidth" class="form-control">
              <option :value="0">0px</option>
              <option :value="1">1px</option>
              <option :value="2">2px</option>
              <option :value="3">3px</option>
              <option :value="4">4px</option>
              <option :value="5">5px</option>
            </select>
          </div>
        </div>
      </div>

      <div class="section-title" style="margin-top: 16px;">Metadata</div>

      <div class="form-group">
        <label>Notes</label>
        <textarea v-model="formData.notes" class="form-control textarea-control" rows="3" placeholder="Add notes..."></textarea>
      </div>

      <div class="form-group">
        <label>URL</label>
        <input v-model="formData.url" type="url" class="form-control" placeholder="https://...">
      </div>

      <div class="form-group">
        <label>Tags</label>
        <div class="tags-input-wrapper">
          <input
            v-model="newTag"
            @keypress.enter.prevent="addTag"
            type="text"
            class="form-control"
            placeholder="Type tag and press Enter"
          >
        </div>
        <div v-if="formData.tags.length > 0" class="tags-list">
          <span v-for="(tag, index) in formData.tags" :key="index" class="tag-chip">
            {{ tag }}
            <button @click="removeTag(index)" class="tag-remove">×</button>
          </span>
        </div>
      </div>

      <div class="form-group">
        <label class="checkbox-inline lock-checkbox">
          <input v-model="formData.locked" type="checkbox">
          <span>🔒 Lock element (read-only)</span>
        </label>
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
  width: 580px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.25);
  z-index: 1000;
  display: flex;
  flex-direction: column;
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
  padding: 12px;
  max-height: 65vh;
  overflow-y: auto;
}

.two-column {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}

.section-title {
  font-size: 12px;
  font-weight: 600;
  color: #374151;
  margin-bottom: 8px;
  padding-bottom: 3px;
  border-bottom: 1px solid #e5e7eb;
}

.form-group {
  margin-bottom: 8px;
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

.color-with-checkbox {
  display: flex;
  align-items: center;
  gap: 6px;
}

.color-with-checkbox .color-input {
  flex: 1;
}

.checkbox-inline {
  display: flex;
  align-items: center;
  gap: 3px;
  font-size: 11px;
  color: #4b5563;
  cursor: pointer;
  white-space: nowrap;
}

.checkbox-inline input[type="checkbox"] {
  cursor: pointer;
}

.button-group {
  display: flex;
  gap: 4px;
}

.style-btn {
  flex: 1;
  padding: 5px;
  border: 1px solid #d1d5db;
  border-radius: 3px;
  background: white;
  cursor: pointer;
  font-size: 13px;
  transition: all 0.2s;
}

.style-btn:hover {
  background: #f3f4f6;
}

.style-btn.active {
  background: #dbeafe;
  border-color: #3b82f6;
  color: #1e40af;
}

.action-btn {
  width: 100%;
  padding: 6px 10px;
  border: 1px solid #d1d5db;
  border-radius: 3px;
  background: white;
  cursor: pointer;
  font-size: 11px;
  text-align: left;
  margin-bottom: 4px;
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
  background: #16a34a;
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
  background: #15803d;
}

.textarea-control {
  resize: vertical;
  min-height: 60px;
  font-family: inherit;
}

.tags-input-wrapper {
  margin-bottom: 6px;
}

.tags-list {
  display: flex;
  flex-wrap: wrap;
  gap: 4px;
  margin-top: 6px;
}

.tag-chip {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 3px 8px;
  background: #e0e7ff;
  color: #3730a3;
  border-radius: 12px;
  font-size: 11px;
  font-weight: 500;
}

.tag-remove {
  background: none;
  border: none;
  color: #3730a3;
  cursor: pointer;
  padding: 0;
  font-size: 16px;
  line-height: 1;
  font-weight: bold;
}

.tag-remove:hover {
  color: #1e1b4b;
}

.lock-checkbox {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 12px;
  color: #4b5563;
  cursor: pointer;
  padding: 8px;
  background: #fef3c7;
  border-radius: 4px;
  border: 1px solid #fde68a;
}

.lock-checkbox input[type="checkbox"] {
  cursor: pointer;
}
</style>

