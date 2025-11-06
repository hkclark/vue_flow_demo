<script setup>
import { ref, computed } from 'vue'
import { VueFlow, useVueFlow, Position } from '@vue-flow/core'
import { Background } from '@vue-flow/background'
import { Controls } from '@vue-flow/controls'
import { MiniMap } from '@vue-flow/minimap'
import '@vue-flow/core/dist/style.css'
import '@vue-flow/core/dist/theme-default.css'
import '@vue-flow/controls/dist/style.css'
import '@vue-flow/minimap/dist/style.css'
import NetworkNode from './components/NetworkNode.vue'
import DevicePalette from './components/DevicePalette.vue'
import EdgeModal from './components/EdgeModal.vue'
import NodeModal from './components/NodeModal.vue'
import { deviceIcons } from './icons/deviceIcons.js'

const { onConnect, addNodes, addEdges, toObject, fromObject, setViewport, updateNode, updateEdge, removeNodes, removeEdges, getNodes, getEdges } = useVueFlow()

const nodes = ref([])
const edges = ref([])
const selectedConnectionType = ref('ethernet')
const showCiscoDevices = ref(true)
const showBasicShapes = ref(true)
const showEdgeModal = ref(false)
const showNodeModal = ref(false)
const selectedEdge = ref(null)
const selectedNodeId = ref(null)

const nodeTypes = {
  network: NetworkNode
}

const connectionTypes = [
  { type: 'ethernet', label: 'Ethernet', color: '#005073', animated: false, style: {} },
  { type: 'wan', label: 'WAN', color: '#dc3545', animated: false, style: { strokeDasharray: '8,4' } },
  { type: 'wireless', label: 'Wireless', color: '#28a745', animated: true, style: { strokeDasharray: '2,4' } },
  { type: 'fiber', label: 'Fiber', color: '#ffc107', animated: false, style: {} }
]

const devicePalette = [
  { type: 'l2_switch', label: 'L2 Switch', svg: deviceIcons.l2_switch, isIcon: true },
  { type: 'l3_switch', label: 'L3 Switch', svg: deviceIcons.l3_switch, isIcon: true },
  { type: 'router', label: 'Router', svg: deviceIcons.router, isIcon: true },
  { type: 'firewall', label: 'Firewall', svg: deviceIcons.firewall, isIcon: true },
  { type: 'access_point', label: 'Access Point', svg: deviceIcons.access_point, isIcon: true },
  { type: 'workstation', label: 'Workstation', svg: deviceIcons.workstation, isIcon: true },
  { type: 'server', label: 'Server', svg: deviceIcons.server, isIcon: true },
  { type: 'printer', label: 'Printer', svg: deviceIcons.printer, isIcon: true },
  { type: 'cloud', label: 'Cloud', svg: deviceIcons.cloud, isIcon: true }
]

const shapePalette = [
  { type: 'text', label: 'Text', svg: '<text x="25" y="30" text-anchor="middle" font-size="16" fill="#005073">Text</text>', isIcon: false },
  { type: 'circle', label: 'Circle', svg: '<circle cx="25" cy="25" r="20" fill="#f0f0f0" stroke="#005073" stroke-width="2"/>', isIcon: false },
  { type: 'oval', label: 'Oval', svg: '<ellipse cx="25" cy="25" rx="23" ry="15" fill="#f0f0f0" stroke="#005073" stroke-width="2"/>', isIcon: false },
  { type: 'square', label: 'Square', svg: '<rect x="5" y="5" width="40" height="40" fill="#f0f0f0" stroke="#005073" stroke-width="2"/>', isIcon: false },
  { type: 'rectangle', label: 'Rectangle', svg: '<rect x="5" y="15" width="40" height="20" fill="#f0f0f0" stroke="#005073" stroke-width="2"/>', isIcon: false }
]

onConnect((params) => {
  const connType = connectionTypes.find(ct => ct.type === selectedConnectionType.value)

  const edge = {
    ...params,
    type: 'smoothstep',
    animated: connType.animated,
    style: {
      stroke: connType.color,
      strokeWidth: 2,
      ...connType.style
    },
    data: {
      connectionType: selectedConnectionType.value,
      centerLabel: '',
      sourceLabel: '',
      targetLabel: '',
      strokeWidth: 2,
      strokeColor: connType.color,
      lineStyle: connType.style.strokeDasharray || '',
      labelFontSize: 12,
      labelColor: connType.color,
      sourceMarker: 'none',
      targetMarker: 'none'
    }
  }

  addEdges([edge])
})

const onDrop = (event) => {
  event.preventDefault()

  const data = event.dataTransfer.getData('application/json')
  if (!data) return

  const item = JSON.parse(data)
  const { left, top } = event.currentTarget.getBoundingClientRect()

  const position = {
    x: event.clientX - left - 35,
    y: event.clientY - top - 35
  }

  // Text elements have different defaults
  const isText = item.type === 'text'

  const newNode = {
    id: `node-${Date.now()}`,
    type: 'network',
    position,
    data: {
      label: item.label,
      svg: item.svg,
      deviceType: item.type,
      isIcon: item.isIcon,
      width: 70,
      height: 70,
      labelPosition: 'bottom',
      labelColor: '#495057',
      labelFontSize: 11,
      labelBold: false,
      labelItalic: false,
      labelUnderline: false,
      fillColor: isText ? 'none' : '#ffffff',
      borderColor: '#005073',
      borderWidth: isText ? 0 : 2
    },
    sourcePosition: Position.Right,
    targetPosition: Position.Left,
    zIndex: nodes.value.length
  }

  addNodes([newNode])
}

const onDragOver = (event) => {
  event.preventDefault()
  event.dataTransfer.dropEffect = 'move'
}

const onEdgeClick = (event) => {
  // Close any open node modal first
  showNodeModal.value = false
  selectedNodeId.value = null

  // event.edge contains the edge data in Vue Flow
  const edge = event.edge || event
  if (edge && edge.id) {
    selectedEdge.value = edge
    showEdgeModal.value = true
  }
}

const onNodeClick = (event) => {
  // Close any open edge modal first
  showEdgeModal.value = false
  selectedEdge.value = null

  // event.node contains the node data in Vue Flow
  const node = event.node || event
  if (node && node.id) {
    // Don't change z-order, just open modal
    selectedNodeId.value = node.id
    showNodeModal.value = true
  }
}

const updateEdgeData = (edgeData) => {
  const edgeIndex = edges.value.findIndex(e => e.id === edgeData.id)
  if (edgeIndex === -1) return

  const lineStyle = edgeData.lineStyle === 'solid' ? '' :
                   edgeData.lineStyle === 'dashed' ? '8,4' : '2,4'

  const markerStart = edgeData.sourceMarker !== 'none' ? {
    type: edgeData.sourceMarker,
    color: edgeData.strokeColor,
    width: 20,
    height: 20
  } : undefined

  const markerEnd = edgeData.targetMarker !== 'none' ? {
    type: edgeData.targetMarker,
    color: edgeData.strokeColor,
    width: 20,
    height: 20
  } : undefined

  // Update the edge in the array directly
  edges.value[edgeIndex] = {
    ...edges.value[edgeIndex],
    type: 'smoothstep',
    style: {
      stroke: edgeData.strokeColor,
      strokeWidth: edgeData.strokeWidth,
      strokeDasharray: lineStyle
    },
    animated: edgeData.lineStyle === 'dotted',
    label: edgeData.centerLabel,
    labelStyle: {
      fill: edgeData.labelColor,
      fontWeight: 600,
      fontSize: edgeData.labelFontSize
    },
    labelBgStyle: { fill: 'white', fillOpacity: 0.8 },
    labelShowBg: true,
    markerStart: markerStart,
    markerEnd: markerEnd,
    data: {
      ...edgeData,
      // Store labels for custom rendering
      sourceLabel: edgeData.sourceLabel,
      targetLabel: edgeData.targetLabel
    }
  }

  // Force reactivity
  edges.value = [...edges.value]

  showEdgeModal.value = false
  selectedEdge.value = null
}

const sendEdgeToFront = (edgeId) => {
  const edgeIndex = edges.value.findIndex(e => e.id === edgeId)
  if (edgeIndex === -1) return

  const maxZIndex = Math.max(...edges.value.map(e => e.zIndex || 0), ...nodes.value.map(n => n.zIndex || 0), 0)

  edges.value[edgeIndex] = {
    ...edges.value[edgeIndex],
    zIndex: maxZIndex + 1
  }

  edges.value = [...edges.value]
  showEdgeModal.value = false
}

const sendEdgeToBack = (edgeId) => {
  const edgeIndex = edges.value.findIndex(e => e.id === edgeId)
  if (edgeIndex === -1) return

  const minZIndex = Math.min(...edges.value.map(e => e.zIndex || 0), ...nodes.value.map(n => n.zIndex || 0), 0)

  edges.value[edgeIndex] = {
    ...edges.value[edgeIndex],
    zIndex: minZIndex - 1
  }

  edges.value = [...edges.value]
  showEdgeModal.value = false
}

const updateNodeData = (nodeData) => {
  const node = nodes.value.find(n => n.id === nodeData.id)
  if (!node) return

  updateNode(nodeData.id, {
    data: nodeData
  })

  showNodeModal.value = false
}

const handleNodeDataUpdate = (nodeId, newData) => {
  updateNode(nodeId, {
    data: newData
  })
}

const duplicateNode = (nodeId) => {
  const node = nodes.value.find(n => n.id === nodeId)
  if (!node) return

  const maxZIndex = Math.max(...nodes.value.map(n => n.zIndex || 0), 0)

  const newNode = {
    id: `node-${Date.now()}-${Math.random()}`,
    type: node.type,
    position: {
      x: node.position.x + 50,
      y: node.position.y + 50
    },
    data: {
      ...node.data
    },
    sourcePosition: node.sourcePosition,
    targetPosition: node.targetPosition,
    zIndex: maxZIndex + 1
  }

  addNodes([newNode])
  showNodeModal.value = false
}

const sendToFront = (nodeId) => {
  const nodeIndex = nodes.value.findIndex(n => n.id === nodeId)
  if (nodeIndex === -1) return

  const maxZIndex = Math.max(...nodes.value.map(n => n.zIndex || 0), 0)

  updateNode(nodeId, {
    zIndex: maxZIndex + 1
  })
}

const sendToBack = (nodeId) => {
  const nodeIndex = nodes.value.findIndex(n => n.id === nodeId)
  if (nodeIndex === -1) return

  const minZIndex = Math.min(...nodes.value.map(n => n.zIndex || 0), 0)

  updateNode(nodeId, {
    zIndex: minZIndex - 1
  })
}

const deleteNode = (nodeId) => {
  if (confirm('Delete this element?')) {
    removeNodes([nodeId])
    showNodeModal.value = false
  }
}

const saveDiagram = () => {
  const flow = toObject()
  const dataStr = JSON.stringify(flow, null, 2)
  const blob = new Blob([dataStr], { type: 'application/json' })
  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')
  link.href = url
  link.download = `network-topology-${Date.now()}.json`
  link.click()
  URL.revokeObjectURL(url)
}

const loadDiagram = (event) => {
  const file = event.target.files[0]
  if (!file) return

  const reader = new FileReader()
  reader.onload = (e) => {
    try {
      const flow = JSON.parse(e.target.result)
      nodes.value = flow.nodes || []
      edges.value = flow.edges || []
      if (flow.viewport) {
        setViewport(flow.viewport)
      }
    } catch (error) {
      alert('Error loading diagram: ' + error.message)
    }
  }
  reader.readAsText(file)
  event.target.value = ''
}

const clearDiagram = () => {
  if (confirm('Are you sure you want to clear the topology?')) {
    nodes.value = []
    edges.value = []
  }
}

const fileInputRef = ref(null)

const getSourceLabelStyle = (edge) => {
  const sourceNode = nodes.value.find(n => n.id === edge.source)
  if (!sourceNode) return { display: 'none' }

  return {
    position: 'absolute',
    left: `${sourceNode.position.x + (sourceNode.data.width || 70) + 10}px`,
    top: `${sourceNode.position.y + (sourceNode.data.height || 70) / 2}px`,
    transform: 'translateY(-50%)',
    color: edge.data?.labelColor || '#005073',
    fontSize: `${edge.data?.labelFontSize || 12}px`,
    fontWeight: '600',
    background: 'white',
    padding: '2px 6px',
    borderRadius: '3px',
    boxShadow: '0 1px 3px rgba(0,0,0,0.2)',
    whiteSpace: 'nowrap',
    pointerEvents: 'none',
    zIndex: 1000
  }
}

const getTargetLabelStyle = (edge) => {
  const targetNode = nodes.value.find(n => n.id === edge.target)
  if (!targetNode) return { display: 'none' }

  return {
    position: 'absolute',
    left: `${targetNode.position.x - 10}px`,
    top: `${targetNode.position.y + (targetNode.data.height || 70) / 2}px`,
    transform: 'translate(-100%, -50%)',
    color: edge.data?.labelColor || '#005073',
    fontSize: `${edge.data?.labelFontSize || 12}px`,
    fontWeight: '600',
    background: 'white',
    padding: '2px 6px',
    borderRadius: '3px',
    boxShadow: '0 1px 3px rgba(0,0,0,0.2)',
    whiteSpace: 'nowrap',
    pointerEvents: 'none',
    zIndex: 1000
  }
}

</script>

<template>
  <div class="topology-builder">
    <div class="toolbar">
      <div class="toolbar-left">
        <div class="logo">
          <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="#005073" stroke-width="2">
            <circle cx="12" cy="12" r="3"/>
            <line x1="12" y1="3" x2="12" y2="9"/>
            <line x1="12" y1="15" x2="12" y2="21"/>
            <line x1="3" y1="12" x2="9" y2="12"/>
            <line x1="15" y1="12" x2="21" y2="12"/>
          </svg>
          <span class="app-title">Topology Builder</span>
        </div>
      </div>
      <div class="toolbar-right">
        <button @click="clearDiagram" class="btn btn-secondary" title="Clear Topology">
          🗑️ Clear
        </button>
        <button @click="$refs.fileInput.click()" class="btn btn-secondary" title="Import Topology">
          📂 Import
        </button>
        <button @click="saveDiagram" class="btn btn-primary" title="Export Topology">
          💾 Export
        </button>
        <input
          ref="fileInput"
          type="file"
          accept=".json"
          @change="loadDiagram"
          style="display: none"
        />
      </div>
    </div>

    <div class="main-content">
      <DevicePalette
        :devicePalette="devicePalette"
        :shapePalette="shapePalette"
        :connectionTypes="connectionTypes"
        :selectedConnectionType="selectedConnectionType"
        :showCiscoDevices="showCiscoDevices"
        :showBasicShapes="showBasicShapes"
        @update:selectedConnectionType="selectedConnectionType = $event"
        @update:showCiscoDevices="showCiscoDevices = $event"
        @update:showBasicShapes="showBasicShapes = $event"
      />

      <div class="canvas-area">
        <VueFlow
          v-model:nodes="nodes"
          v-model:edges="edges"
          :node-types="nodeTypes"
          @drop="onDrop"
          @dragover="onDragOver"
          @edge-click="onEdgeClick"
          @node-click="onNodeClick"
          @node-data-update="handleNodeDataUpdate"
          class="vue-flow-canvas"
          :default-zoom="1"
          :min-zoom="0.25"
          :max-zoom="4"
        >
          <Background
            pattern-color="#d1d5db"
            :gap="20"
            :size="1"
            variant="lines"
          />
          <Controls
            :show-zoom="true"
            :show-fit-view="true"
            :show-interactive="true"
          />
          <MiniMap
            node-color="#005073"
            mask-color="rgba(0, 0, 0, 0.1)"
          />

          <!-- Render source/target labels as overlays -->
          <div class="edge-labels-overlay">
            <div v-for="edge in edges.filter(e => e.data?.sourceLabel || e.data?.targetLabel)"
                 :key="edge.id"
                 class="edge-label-container">
              <div v-if="edge.data?.sourceLabel && edge.source"
                   class="edge-source-label"
                   :style="getSourceLabelStyle(edge)">
                {{ edge.data.sourceLabel }}
              </div>
              <div v-if="edge.data?.targetLabel && edge.target"
                   class="edge-target-label"
                   :style="getTargetLabelStyle(edge)">
                {{ edge.data.targetLabel }}
              </div>
            </div>
          </div>
        </VueFlow>
      </div>
    </div>

    <EdgeModal
      v-if="showEdgeModal && selectedEdge"
      :edge="selectedEdge"
      @close="showEdgeModal = false"
      @update="updateEdgeData"
      @toFront="sendEdgeToFront"
      @toBack="sendEdgeToBack"
    />

    <NodeModal
      v-if="showNodeModal && selectedNodeId"
      :node="nodes.find(n => n.id === selectedNodeId)"
      @close="showNodeModal = false"
      @update="updateNodeData"
      @duplicate="duplicateNode"
      @toFront="sendToFront"
      @toBack="sendToBack"
      @delete="deleteNode"
    />
  </div>
</template>

<style scoped>
.topology-builder {
  display: flex;
  flex-direction: column;
  width: 100vw;
  height: 100vh;
  background-color: #f8f9fa;
}

.toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: white;
  border-bottom: 1px solid #dee2e6;
  padding: 12px 20px;
  height: 56px;
}

.toolbar-left {
  display: flex;
  align-items: center;
  gap: 16px;
}

.logo {
  display: flex;
  align-items: center;
  gap: 8px;
}

.app-title {
  font-size: 18px;
  font-weight: 600;
  color: #212529;
}

.toolbar-right {
  display: flex;
  gap: 8px;
}

.btn {
  padding: 8px 16px;
  border: 1px solid #dee2e6;
  border-radius: 4px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
  background: white;
  color: #495057;
}

.btn:hover {
  background: #f8f9fa;
}

.btn-primary {
  background: #0d6efd;
  color: white;
  border-color: #0d6efd;
}

.btn-primary:hover {
  background: #0b5ed7;
  border-color: #0b5ed7;
}

.btn-secondary {
  background: white;
  color: #6c757d;
}

.main-content {
  display: flex;
  flex: 1;
  overflow: hidden;
}

.canvas-area {
  flex: 1;
  position: relative;
  background: #f8f9fa;
}

.vue-flow-canvas {
  width: 100%;
  height: 100%;
}
</style>

