<template>
  <div class="header">

  </div>
  <div class="container">
    <div class="container-items">
      <div class="item" @click="onAdd">Condition</div>
      <div class="item">Condition</div>
    </div>
    <div class="container-editor">
      <VueFlow >
        
    
        <Panel :position="PanelPosition.TopRight">
          <div>
            <label for="ishidden">
              hidden
              <input id="ishidden" v-model="isHidden" type="checkbox" />
            </label>
          </div>
        </Panel>
      </VueFlow>
    </div>
    <div class="container-content-element">

    </div>
  </div>
</template>
<script setup>
import '@vue-flow/core/dist/style.css';
import '@vue-flow/core/dist/theme-default.css';
import { createApp } from 'vue'

import { Panel, PanelPosition, VueFlow, useVueFlow} from '@vue-flow/core'
import { ref, watch } from 'vue'

const isHidden = ref(false)

const { nodes, addNodes, setNodes, setEdges, dimensions, setTransform, toObject} = useVueFlow({
  nodes: [
    { id: '1', type: 'input', label: 'Node 1', position: { x: 250, y: 5 } },
    { id: '2', label: 'Node 2', position: { x: 100, y: 100 } },
    { id: '3', label: 'Node 3', position: { x: 400, y: 100 } },
    { id: '4', label: 'Node 4', position: { x: 400, y: 200 } },
  ],
  edges: [
    { id: 'e1-2', source: '1', target: '2' },
    { id: 'e1-3', source: '1', target: '3' },
    { id: 'e3-4', source: '3', target: '4' },
  ],
})

watch(isHidden, () => {
  nodes.value.forEach((n) => (n.hidden = isHidden.value))
  edges.value.forEach((e) => (e.hidden = isHidden.value))
})

const onAdd = () => {
  const id = nodes.value.length + 1

  const newNode = {
    id: `random_node-${id}`,
    label: `Node ${id}`,
    position: { x: Math.random() * dimensions.value.width, y: Math.random() * dimensions.value.height },
  }

  addNodes([newNode])
}
const app = createApp({
  data() {
    return {
    
    }
  }
})

app.mount('#app')
</script>
