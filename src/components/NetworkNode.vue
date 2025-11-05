<script setup>
import { Handle, Position } from '@vue-flow/core'
import { computed, ref } from 'vue'
import { useVueFlow } from '@vue-flow/core'

const props = defineProps({
  data: Object,
  id: String,
  selected: Boolean
})

const { updateNode } = useVueFlow()

const isResizing = ref(false)

const labelStyle = computed(() => ({
  fontSize: `${props.data.labelFontSize}px`,
  color: props.data.labelColor,
  fontWeight: props.data.labelBold ? 'bold' : 'normal',
  fontStyle: props.data.labelItalic ? 'italic' : 'normal',
  textDecoration: props.data.labelUnderline ? 'underline' : 'none'
}))

const nodeStyle = computed(() => ({
  width: `${props.data.width}px`,
  height: `${props.data.height}px`,
  backgroundColor: props.data.fillColor === 'none' ? 'transparent' : props.data.fillColor,
  border: `${props.data.borderWidth}px solid ${props.data.borderColor}`
}))

const iconSize = computed(() => ({
  width: `${props.data.width * 0.8}px`,
  height: `${props.data.height * 0.8}px`
}))

const labelPositionClass = computed(() => `label-${props.data.labelPosition}`)

const startResize = (e, direction) => {
  e.stopPropagation()
  e.preventDefault()
  isResizing.value = true
  
  const startX = e.clientX
  const startY = e.clientY
  const startWidth = props.data.width
  const startHeight = props.data.height
  
  const handleMouseMove = (e) => {
    const deltaX = e.clientX - startX
    const deltaY = e.clientY - startY
    
    let newWidth = startWidth
    let newHeight = startHeight
    
    if (direction.includes('e')) {
      newWidth = Math.max(40, startWidth + deltaX)
    }
    if (direction.includes('s')) {
      newHeight = Math.max(40, startHeight + deltaY)
    }
    if (direction.includes('w')) {
      newWidth = Math.max(40, startWidth - deltaX)
    }
    if (direction.includes('n')) {
      newHeight = Math.max(40, startHeight - deltaY)
    }
    
    updateNode(props.id, {
      data: {
        ...props.data,
        width: newWidth,
        height: newHeight
      }
    })
  }
  
  const handleMouseUp = () => {
    isResizing.value = false
    document.removeEventListener('mousemove', handleMouseMove)
    document.removeEventListener('mouseup', handleMouseUp)
  }
  
  document.addEventListener('mousemove', handleMouseMove)
  document.addEventListener('mouseup', handleMouseUp)
}
</script>

<template>
  <div class="network-node-container">
    <div class="network-node" :style="nodeStyle">
      <!-- Resize handles -->
      <div v-if="selected" class="resize-handles">
        <div class="resize-handle nw" @mousedown="startResize($event, 'nw')"></div>
        <div class="resize-handle n" @mousedown="startResize($event, 'n')"></div>
        <div class="resize-handle ne" @mousedown="startResize($event, 'ne')"></div>
        <div class="resize-handle e" @mousedown="startResize($event, 'e')"></div>
        <div class="resize-handle se" @mousedown="startResize($event, 'se')"></div>
        <div class="resize-handle s" @mousedown="startResize($event, 's')"></div>
        <div class="resize-handle sw" @mousedown="startResize($event, 'sw')"></div>
        <div class="resize-handle w" @mousedown="startResize($event, 'w')"></div>
      </div>
      
      <Handle type="target" :position="Position.Left" />
      <Handle type="target" :position="Position.Top" />
      
      <div class="node-content" v-if="data.deviceType !== 'text'">
        <div class="node-icon" :style="iconSize" v-html="data.svg"></div>
      </div>
      
      <div v-if="data.deviceType === 'text'" class="text-content" :style="labelStyle">
        {{ data.label }}
      </div>
      
      <Handle type="source" :position="Position.Right" />
      <Handle type="source" :position="Position.Bottom" />
    </div>
    
    <!-- Label outside unless center position -->
    <div v-if="data.deviceType !== 'text' && data.labelPosition !== 'center'" 
         class="node-label-external" 
         :class="`label-${data.labelPosition}`"
         :style="labelStyle">
      {{ data.label }}
    </div>
    
    <!-- Center label overlays on the icon -->
    <div v-if="data.deviceType !== 'text' && data.labelPosition === 'center'" 
         class="node-label-center"
         :style="labelStyle">
      {{ data.label }}
    </div>
  </div>
</template>

<style scoped>
.network-node-container {
  position: relative;
  display: inline-block;
}

.network-node {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: white;
  border-radius: 4px;
  position: relative;
}

.resize-handles {
  position: absolute;
  top: -4px;
  left: -4px;
  right: -4px;
  bottom: -4px;
  pointer-events: none;
  z-index: 10;
}

.resize-handle {
  position: absolute;
  background: #0d6efd;
  border: 1px solid white;
  pointer-events: all;
  z-index: 11;
}

.resize-handle.nw,
.resize-handle.ne,
.resize-handle.sw,
.resize-handle.se {
  width: 8px;
  height: 8px;
  border-radius: 50%;
}

.resize-handle.n,
.resize-handle.s {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  left: 50%;
  transform: translateX(-50%);
}

.resize-handle.e,
.resize-handle.w {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  top: 50%;
  transform: translateY(-50%);
}

.resize-handle.nw {
  top: -4px;
  left: -4px;
  cursor: nw-resize;
}

.resize-handle.n {
  top: -4px;
  cursor: n-resize;
}

.resize-handle.ne {
  top: -4px;
  right: -4px;
  cursor: ne-resize;
}

.resize-handle.e {
  right: -4px;
  cursor: e-resize;
}

.resize-handle.se {
  bottom: -4px;
  right: -4px;
  cursor: se-resize;
}

.resize-handle.s {
  bottom: -4px;
  cursor: s-resize;
}

.resize-handle.sw {
  bottom: -4px;
  left: -4px;
  cursor: sw-resize;
}

.resize-handle.w {
  left: -4px;
  cursor: w-resize;
}

.node-content {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
  padding: 4px;
}

.node-icon {
  display: flex;
  align-items: center;
  justify-content: center;
}

.node-icon :deep(svg) {
  width: 100%;
  height: 100%;
}

.node-label-external {
  position: absolute;
  text-align: center;
  word-break: break-word;
  max-width: 150px;
  white-space: nowrap;
  pointer-events: none;
}

.node-label-external.label-top {
  bottom: 100%;
  left: 50%;
  transform: translateX(-50%);
  margin-bottom: 4px;
}

.node-label-external.label-bottom {
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
  margin-top: 4px;
}

.node-label-external.label-left {
  right: 100%;
  top: 50%;
  transform: translateY(-50%);
  margin-right: 8px;
}

.node-label-external.label-right {
  left: 100%;
  top: 50%;
  transform: translateY(-50%);
  margin-left: 8px;
}

.node-label-center {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
  word-break: break-word;
  pointer-events: none;
  z-index: 5;
}

.text-content {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  padding: 8px;
  word-break: break-word;
}
</style>

<style scoped>
.network-node {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: white;
  border-radius: 4px;
  position: relative;
}

.resize-handles {
  position: absolute;
  top: -4px;
  left: -4px;
  right: -4px;
  bottom: -4px;
  pointer-events: none;
}

.resize-handle {
  position: absolute;
  background: #0d6efd;
  border: 1px solid white;
  pointer-events: all;
}

.resize-handle.nw,
.resize-handle.ne,
.resize-handle.sw,
.resize-handle.se {
  width: 8px;
  height: 8px;
  border-radius: 50%;
}

.resize-handle.n,
.resize-handle.s {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  left: 50%;
  transform: translateX(-50%);
}

.resize-handle.e,
.resize-handle.w {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  top: 50%;
  transform: translateY(-50%);
}

.resize-handle.nw {
  top: -4px;
  left: -4px;
  cursor: nw-resize;
}

.resize-handle.n {
  top: -4px;
  cursor: n-resize;
}

.resize-handle.ne {
  top: -4px;
  right: -4px;
  cursor: ne-resize;
}

.resize-handle.e {
  right: -4px;
  cursor: e-resize;
}

.resize-handle.se {
  bottom: -4px;
  right: -4px;
  cursor: se-resize;
}

.resize-handle.s {
  bottom: -4px;
  cursor: s-resize;
}

.resize-handle.sw {
  bottom: -4px;
  left: -4px;
  cursor: sw-resize;
}

.resize-handle.w {
  left: -4px;
  cursor: w-resize;
}

.node-wrapper {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
  padding: 8px;
}

.node-wrapper.label-bottom {
  flex-direction: column;
}

.node-wrapper.label-top {
  flex-direction: column-reverse;
}

.node-wrapper.label-left {
  flex-direction: row-reverse;
}

.node-wrapper.label-right {
  flex-direction: row;
}

.node-wrapper.label-center {
  flex-direction: column;
}

.node-content {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.node-icon {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.node-icon :deep(svg) {
  max-width: 90%;
  max-height: 90%;
}

.node-label {
  margin-top: 4px;
  text-align: center;
  word-break: break-word;
  max-width: 120px;
}

.label-top .node-label,
.label-bottom .node-label {
  margin: 4px 0;
}

.label-left .node-label,
.label-right .node-label {
  margin: 0 4px;
}

.label-center .node-label {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.text-content {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  padding: 8px;
  word-break: break-word;
}
</style>

