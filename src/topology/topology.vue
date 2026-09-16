<script setup lang="ts">
import { computed, ref, shallowRef, triggerRef } from 'vue'
import { VueFlow, useVueFlow } from '@vue-flow/core'
import type { Connection, Edge, EdgeMouseEvent, Node, NodeMouseEvent } from '@vue-flow/core'
import TopologyNode from './TopologyNode.vue'
import TestCenterNode from './TestCenterNode.vue'

import '@vue-flow/core/dist/style.css'
import '@vue-flow/core/dist/theme-default.css'

type DeviceKind = 'switch' | 'testcenter'
interface Port { id: string; name: string; speed: string; pid: string; side: 'left' | 'right' }
interface DeviceData { name: string; ip: string; protocolPort?: string; username?: string; password?: string; version?: string; ports: Port[] }
type TopologyNode = Node<DeviceData>
type ContextMenu = { x: number; y: number; kind: 'node' | 'port' | 'edge'; nodeId?: string; portId?: string; edgeId?: string }

const nodes = shallowRef<TopologyNode[]>([
  { id: 'switch-1', type: 'switch', position: { x: 180, y: 150 }, data: { name: 'Core Switch', ip: '10.0.0.10', protocolPort: '22', username: 'admin', password: '', ports: [{ id: 'ge-1', name: 'GE 1', speed: '1G', pid: '1', side: 'right' }, { id: 'ge-2', name: 'GE 2', speed: '1G', pid: '2', side: 'right' }] } },
  { id: 'testcenter-1', type: 'testcenter', position: { x: 560, y: 150 }, data: { name: 'Spirent TestCenter', ip: '10.0.0.20', version: '5.50', ports: [{ id: 'port-1', name: 'Port 1', speed: '10G', pid: '1', side: 'left' }] } },
])
const edges = shallowRef<Edge[]>([])
const selectedId = ref<string | null>('switch-1')
const selectedPort = ref<Port | null>(null)
const portDraft = ref<Port | null>(null)
const contextMenu = ref<ContextMenu | null>(null)
const xmlFileInput = ref<HTMLInputElement | null>(null)
const { addEdges, screenToFlowCoordinate } = useVueFlow()

const selectedNode = computed<TopologyNode | null>(() => {
  const node = nodes.value.find((item) => item.id === selectedId.value)
  return node ?? null
})
const selectedData = computed<DeviceData | undefined>(() => selectedNode.value?.data)

function selectNode(event: NodeMouseEvent) {
  selectedId.value = event.node.id
  selectedPort.value = null
  contextMenu.value = null
}

function openNodeMenu(event: NodeMouseEvent) {
  selectedId.value = event.node.id
  const mouseEvent = event.event
  if (mouseEvent instanceof MouseEvent) contextMenu.value = { x: mouseEvent.clientX, y: mouseEvent.clientY, kind: 'node', nodeId: event.node.id }
}

function openPortMenu(port: Port, event: MouseEvent, nodeId: string) {
  selectedId.value = nodeId
  contextMenu.value = { x: event.clientX, y: event.clientY, kind: 'port', nodeId, portId: port.id }
}

function openEdgeMenu(event: EdgeMouseEvent) {
  if (event.event instanceof MouseEvent) contextMenu.value = { x: event.event.clientX, y: event.event.clientY, kind: 'edge', edgeId: event.edge.id }
}

function deleteContextItem() {
  const menu = contextMenu.value
  if (!menu) return
  if (menu.kind === 'node' && menu.nodeId) {
    nodes.value = nodes.value.filter((node) => node.id !== menu.nodeId)
    edges.value = edges.value.filter((edge) => edge.source !== menu.nodeId && edge.target !== menu.nodeId)
    if (selectedId.value === menu.nodeId) selectedId.value = null
  } else if (menu.kind === 'edge' && menu.edgeId) {
    edges.value = edges.value.filter((edge) => edge.id !== menu.edgeId)
  } else if (menu.kind === 'port' && menu.nodeId && menu.portId) {
    const node = nodes.value.find((item) => item.id === menu.nodeId)
    if (node?.data) node.data.ports = node.data.ports.filter((port) => port.id !== menu.portId)
    triggerRef(nodes)
    edges.value = edges.value.filter((edge) => edge.sourceHandle !== menu.portId && edge.targetHandle !== menu.portId)
  }
  selectedPort.value = null
  portDraft.value = null
  contextMenu.value = null
}

function closeContextMenu() {
  contextMenu.value = null
}

function onConnect(connection: Connection) {
  if (connection.source && connection.target) addEdges(connection)
}

function dragStart(event: DragEvent, kind: DeviceKind | 'port') {
  event.dataTransfer?.setData('application/topology-item', kind)
  if (event.dataTransfer) event.dataTransfer.effectAllowed = 'copy'
}

function onDrop(event: DragEvent) {
  const kind = event.dataTransfer?.getData('application/topology-item')
  if (!kind) return
  const position = screenToFlowCoordinate({ x: event.clientX, y: event.clientY })
  if (kind === 'port') {
    const target = nodes.value.find((node) => position.x >= node.position.x && position.x <= node.position.x + 250 && position.y >= node.position.y && position.y <= node.position.y + 180)
    if (target) addPort(target.id, position.x < target.position.x + 125 ? 'left' : 'right')
    return
  }
  const id = `${kind}-${Date.now()}`
  const data: DeviceData = kind === 'switch'
    ? { name: 'New Switch', ip: '', protocolPort: '22', username: '', password: '', ports: [] }
    : { name: 'New TestCenter', ip: '', version: '', ports: [] }
  nodes.value = [...nodes.value, { id, type: kind, position, data }]
  selectedId.value = id
}

function addPort(nodeId = selectedId.value, side: 'left' | 'right' = 'right') {
  if (!nodeId) return
  const node = nodes.value.find((item) => item.id === nodeId)
  if (!node?.data) return
  const number = node.data.ports.length + 1
  node.data.ports.push({ id: `port-${Date.now()}`, name: `Port ${number}`, speed: '1G', pid: String(number), side })
  triggerRef(nodes)
}

function editPort(port: Port) {
  selectedPort.value = port
  portDraft.value = { ...port }
}

function savePort() {
  if (!selectedPort.value || !portDraft.value) return
  Object.assign(selectedPort.value, portDraft.value)
  triggerRef(nodes)
  selectedPort.value = null
  portDraft.value = null
}

function escapeXml(value: unknown) {
  return String(value ?? '').replace(/[<>&'"]/g, (character) => ({ '<': '&lt;', '>': '&gt;', '&': '&amp;', "'": '&apos;', '"': '&quot;' })[character] ?? character)
}

function saveXml() {
  const xml = `<?xml version="1.0" encoding="UTF-8"?>\n<topology>\n${nodes.value.map((node) => {
    const data = node.data
    if (!data) return ''
    const attrs = Object.entries(data).filter(([key]) => key !== 'ports').map(([key, value]) => ` ${key}="${escapeXml(value)}"`).join('')
    const ports = data.ports.map((port) => `    <port id="${escapeXml(port.id)}" name="${escapeXml(port.name)}" speed="${escapeXml(port.speed)}" pid="${escapeXml(port.pid)}" side="${escapeXml(port.side)}" />`).join('\n')
    return `  <device id="${escapeXml(node.id)}" type="${escapeXml(node.type)}" x="${node.position.x}" y="${node.position.y}"${attrs}>\n${ports}\n  </device>`
  }).join('\n')}${edges.value.length ? `\n${edges.value.map((edge) => `  <edge id="${escapeXml(edge.id)}" source="${escapeXml(edge.source)}" sourceHandle="${escapeXml(edge.sourceHandle)}" target="${escapeXml(edge.target)}" targetHandle="${escapeXml(edge.targetHandle)}" />`).join('\n')}` : ''}\n</topology>\n`
  const url = URL.createObjectURL(new Blob([xml], { type: 'application/xml' }))
  const anchor = document.createElement('a')
  anchor.href = url
  anchor.download = 'topology.xml'
  anchor.click()
  URL.revokeObjectURL(url)
}

function openXmlPicker() {
  xmlFileInput.value?.click()
}

function attribute(element: Element, name: string, fallback = '') {
  return element.getAttribute(name) ?? fallback
}

function parseNumber(value: string, fallback: number) {
  const number = Number(value)
  return Number.isFinite(number) ? number : fallback
}

function loadXml(event: Event) {
  const input = event.target as HTMLInputElement
  const file = input.files?.[0]
  input.value = ''
  if (!file) return

  const reader = new FileReader()
  reader.onload = () => {
    const document = new DOMParser().parseFromString(String(reader.result), 'application/xml')
    if (document.querySelector('parsererror') || document.documentElement.nodeName !== 'topology') {
      window.alert('The selected file is not a valid topology XML file.')
      return
    }

    const importedNodes: TopologyNode[] = Array.from(document.querySelectorAll(':scope > device')).map((element, index) => {
      const type = attribute(element, 'type', 'switch') === 'testcenter' ? 'testcenter' : 'switch'
      const data: DeviceData = {
        name: attribute(element, 'name', `Imported ${type}`),
        ip: attribute(element, 'ip'),
        ports: Array.from(element.querySelectorAll(':scope > port')).map((port, portIndex) => ({
          id: attribute(port, 'id', `${attribute(element, 'id', `device-${index}`)}-port-${portIndex + 1}`),
          name: attribute(port, 'name', `Port ${portIndex + 1}`),
          speed: attribute(port, 'speed', '1G'),
          pid: attribute(port, 'pid', String(portIndex + 1)),
          side: attribute(port, 'side', 'right') === 'left' ? 'left' : 'right',
        })),
      }
      if (type === 'switch') {
        data.protocolPort = attribute(element, 'protocolPort')
        data.username = attribute(element, 'username')
        data.password = attribute(element, 'password')
      } else {
        data.version = attribute(element, 'version')
      }
      return {
        id: attribute(element, 'id', `device-${index + 1}`),
        type,
        position: {
          x: parseNumber(attribute(element, 'x'), 120 + index * 220),
          y: parseNumber(attribute(element, 'y'), 120),
        },
        data,
      }
    })
    const importedEdges: Edge[] = Array.from(document.querySelectorAll(':scope > edge')).map((element, index) => ({
      id: attribute(element, 'id', `edge-${index + 1}`),
      source: attribute(element, 'source'),
      sourceHandle: attribute(element, 'sourceHandle') || undefined,
      target: attribute(element, 'target'),
      targetHandle: attribute(element, 'targetHandle') || undefined,
    })).filter((edge) => edge.source && edge.target)

    nodes.value = importedNodes
    edges.value = importedEdges
    selectedId.value = importedNodes[0]?.id ?? null
    selectedPort.value = null
    portDraft.value = null
    contextMenu.value = null
  }
  reader.onerror = () => window.alert('The topology XML file could not be read.')
  reader.readAsText(file)
}

function onCanvasDragOver(event: DragEvent) {
  event.preventDefault()
  if (event.dataTransfer) event.dataTransfer.dropEffect = 'copy'
}
</script>

<template>
  <div class="topology-shell">
    <aside class="left-panel">
      <div class="eyebrow">TOPOLOGY BUILDER</div>
      <h1>Network lab</h1>
      <p class="muted">Drag equipment and ports onto the canvas.</p>
      <div class="menu-label">EQUIPMENT</div>
      <div class="palette-card" draggable="true" @dragstart="dragStart($event, 'switch')"><span class="palette-icon switch">S</span><span><strong>Switch</strong><small>Network device</small></span><span class="drag-grip">⋮⋮</span></div>
      <div class="palette-card" draggable="true" @dragstart="dragStart($event, 'testcenter')"><span class="palette-icon testcenter">T</span><span><strong>Spirent TestCenter</strong><small>Traffic generator</small></span><span class="drag-grip">⋮⋮</span></div>
      <div class="menu-label">PORTS</div>
      <div class="palette-card port-card" draggable="true" @dragstart="dragStart($event, 'port')"><span class="port-dot"></span><span><strong>Network port</strong><small>Drop on a device</small></span><span class="drag-grip">⋮⋮</span></div>
      <div class="legend"><span class="legend-dot connected"></span> Connected <span class="legend-dot available"></span> Available</div>
    </aside>

    <main class="canvas-area" @drop="onDrop" @dragover="onCanvasDragOver" @click="closeContextMenu">
      <div class="canvas-toolbar"><span class="status-dot"></span><span>Unsaved topology</span><span class="toolbar-hint">Drag to pan · Scroll to zoom</span></div>
      <VueFlow v-model:nodes="nodes" v-model:edges="edges" :default-viewport="{ zoom: 0.9, x: 30, y: 30 }" fit-view-on-init @connect="onConnect" @node-click="selectNode" @node-context-menu="openNodeMenu" @edge-context-menu="openEdgeMenu">
        <template #node-switch="props"><TopologyNode v-bind="props" @port-dblclick="editPort" @port-contextmenu="openPortMenu" /></template>
        <template #node-testcenter="props"><TestCenterNode v-bind="props" @port-dblclick="editPort" @port-contextmenu="openPortMenu" /></template>
      </VueFlow>
      <div v-if="contextMenu" class="context-menu" :style="{ left: `${contextMenu.x}px`, top: `${contextMenu.y}px` }" @click.stop>
        <button @click="deleteContextItem">Delete {{ contextMenu.kind }}</button>
      </div>
    </main>

    <aside class="right-panel">
      <input ref="xmlFileInput" class="file-input" type="file" accept=".xml,application/xml,text/xml" @change="loadXml" />
      <div class="inspector-header"><div><div class="eyebrow">INSPECTOR</div><h2>{{ selectedData?.name || 'No selection' }}</h2></div><div class="inspector-actions"><button class="icon-button" title="Open topology XML" @click="openXmlPicker">↑</button><button class="icon-button" title="Save topology as XML" @click="saveXml">↓</button></div></div>
      <div v-if="selectedData && !selectedPort" class="form">
        <label>Name<input v-model="selectedData.name" /></label>
        <label>IP address<input v-model="selectedData.ip" placeholder="192.168.1.10" /></label>
        <template v-if="selectedNode?.type === 'switch'">
          <label>Protocol port<input v-model="selectedData.protocolPort" /></label>
          <label>Login username<input v-model="selectedData.username" /></label>
          <label>Password<input v-model="selectedData.password" type="password" /></label>
        </template>
        <label v-else>Version<input v-model="selectedData.version" placeholder="5.50" /></label>
        <button class="add-button" @click="addPort()">＋ Add port</button>
      </div>
      <div v-else-if="selectedPort && portDraft" class="form">
        <div class="section-title">EDIT PORT</div>
        <label>Interface name<input v-model="portDraft.name" /></label>
        <label>Interface type / speed<input v-model="portDraft.speed" placeholder="10G" /></label>
        <label>PID / port ID<input v-model="portDraft.pid" /></label>
        <div class="button-row"><button class="ghost-button" @click="selectedPort = null">Cancel</button><button class="save-button" @click="savePort">Save port</button></div>
      </div>
      <div v-else class="empty-state">Select a device to edit its connection details.</div>
      <div class="save-box"><div><strong>Import or export topology</strong><p>Open a saved XML file or save the current canvas for later.</p></div><div class="export-actions"><button class="open-button" @click="openXmlPicker">Open XML <span>↥</span></button><button class="export-button" @click="saveXml">Save XML <span>↗</span></button></div></div>
    </aside>
  </div>
</template>

<style>
.topology-shell { display: grid; grid-template-columns: 248px 1fr 292px; height: 100vh; overflow: hidden; background: #0b1424; color: #dce8fa; font: 13px Inter, ui-sans-serif, system-ui, sans-serif; }
.left-panel, .right-panel { background: #101d31; padding: 28px 20px; z-index: 2; }
.left-panel { border-right: 1px solid #1e3150; }.right-panel { border-left: 1px solid #1e3150; }
.eyebrow, .menu-label, .section-title { color: #7088aa; font-size: 10px; font-weight: 700; letter-spacing: .14em; }.menu-label { margin: 30px 0 10px; }
h1 { margin: 7px 0; font-size: 21px; letter-spacing: -.03em; }.muted { margin: 0; color: #7187a7; line-height: 1.5; }
.palette-card { display: flex; align-items: center; gap: 10px; min-height: 56px; margin: 8px 0; padding: 10px; border: 1px solid #253b5d; border-radius: 10px; background: #152642; cursor: grab; }.palette-card:active { cursor: grabbing; }
.palette-card strong, .palette-card small { display: block; }.palette-card strong { font-size: 12px; }.palette-card small { margin-top: 4px; color: #7890b2; font-size: 10px; }.palette-icon { display: grid; place-items: center; flex: 0 0 28px; height: 28px; border-radius: 8px; font-weight: 700; }.palette-icon.switch { background: #3472d9; }.palette-icon.testcenter { background: #8e58c2; }.drag-grip { margin-left: auto; color: #627a9c; letter-spacing: -4px; }
.port-dot, .legend-dot { width: 10px; height: 10px; border-radius: 50%; background: #4f8cff; box-shadow: 0 0 0 3px #234371; }.port-card { min-height: 45px; }.port-card .port-dot { width: 8px; height: 8px; box-shadow: 0 0 0 2px #234371; }.legend { display: flex; align-items: center; gap: 7px; margin-top: 28px; color: #748bab; font-size: 10px; }.legend-dot { width: 6px; height: 6px; box-shadow: none; margin-left: 6px; }.legend-dot.available { background: #6a7b94; }
.canvas-area { position: relative; min-width: 0; background-color: #0c1729; background-image: linear-gradient(#182942 1px, transparent 1px), linear-gradient(90deg, #182942 1px, transparent 1px); background-size: 24px 24px; }.canvas-toolbar { position: absolute; top: 18px; left: 24px; z-index: 3; display: flex; align-items: center; gap: 8px; color: #8298b8; font-size: 11px; }.status-dot { width: 7px; height: 7px; border-radius: 50%; background: #e6aa54; }.toolbar-hint { margin-left: 18px; color: #536b8d; }
.context-menu { position: fixed; z-index: 20; min-width: 120px; padding: 5px; border: 1px solid #315174; border-radius: 8px; background: #172b49; box-shadow: 0 10px 24px rgba(0, 0, 0, .35); }.context-menu button { width: 100%; border: 0; border-radius: 5px; padding: 8px 10px; background: transparent; color: #f0f5ff; text-align: left; font: inherit; cursor: pointer; }.context-menu button:hover { background: #a94355; }
.vue-flow { background: #0c1729; }.vue-flow__edge-path { stroke: #6f9ddd; stroke-width: 2; }.vue-flow__edge.selected .vue-flow__edge-path { stroke: #b277ff; }.vue-flow__node.selected > div { box-shadow: 0 0 0 2px #72a9ff, 0 8px 20px rgba(3, 12, 29, .32); }
.file-input { display: none; }.inspector-header { display: flex; justify-content: space-between; align-items: flex-start; padding-bottom: 22px; border-bottom: 1px solid #203552; }.inspector-header h2 { max-width: 195px; margin: 8px 0 0; overflow: hidden; text-overflow: ellipsis; font-size: 17px; white-space: nowrap; }.inspector-actions { display: flex; gap: 6px; }.icon-button { width: 32px; height: 32px; border: 1px solid #315174; border-radius: 8px; background: #172b49; color: #a9c9f5; font-size: 18px; cursor: pointer; }
.form { display: grid; gap: 14px; padding-top: 22px; }.form label { display: grid; gap: 7px; color: #8197b7; font-size: 11px; }.form input { width: 100%; box-sizing: border-box; padding: 9px 10px; border: 1px solid #294261; border-radius: 7px; outline: none; background: #0c192c; color: #e0ebfb; font: inherit; }.form input:focus { border-color: #548ce1; }.add-button, .save-button, .export-button { border: 0; border-radius: 7px; padding: 10px 12px; color: white; background: #3477dc; font-weight: 600; cursor: pointer; }.add-button { margin-top: 5px; background: #1b3558; color: #9fc5f8; }.button-row { display: flex; gap: 8px; }.ghost-button { border: 1px solid #2c4668; border-radius: 7px; padding: 9px 12px; background: transparent; color: #9eb4d2; cursor: pointer; }.save-button { flex: 1; }
.empty-state { padding-top: 25px; color: #7187a7; line-height: 1.5; }.save-box { margin-top: auto; padding-top: 22px; border-top: 1px solid #203552; }.save-box strong { font-size: 12px; }.save-box p { margin: 7px 0 14px; color: #7187a7; font-size: 11px; line-height: 1.5; }.export-actions { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; }.open-button, .export-button { width: 100%; border: 1px solid #315174; border-radius: 7px; padding: 10px 9px; background: #172b49; color: #a9c9f5; text-align: left; font: inherit; cursor: pointer; }.export-button { border: 0; background: #3477dc; color: white; }.open-button span, .export-button span { float: right; font-size: 16px; }
</style>
