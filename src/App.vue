<template>
  <div class="header">
    <h1 style="font-size: 45px">Flow Chart</h1>
  </div>
  <div class="container">
    <div class="container-items">
      <div class="item" @click="onAdd('input')" style="border-bottom: 3px solid #1787DE;">Start</div>
      <div class="item" @click="onAdd('decision')" style="border-bottom: 3px solid green;">Decision</div>
      <div class="item" @click="onAdd('notice')" style="border-bottom: 3px solid orange;">Notice</div>
      <div class="item" @click="onAddCondition()" style="border-bottom: 3px solid gray;">Condition</div>
      <div class="item" @click="onAdd('output')" style="border-bottom: 3px solid red;">End</div>
    </div>
    <div class="container-editor">
      <VueFlow fit-view-on-init v-model="nodes">
        <Background :pattern-color="dark ? '#FFFFFB' : '#aaa'" gap="10" />
    <MiniMap />
    <Controls />
        <Panel :position="PanelPosition.TopRight">
          <div>
              <button @click="clearData" class="clear-button" type="button">Clear</button>
          </div>
        </Panel>
      </VueFlow>
    </div>
    <div class="container-content-element">
      <div class="form-input">
        <label for="id">Id:</label>
        <input type="text" name="id" v-model="idChoosing" disabled>
        <label for="level">Level:</label>
        <input type="text" name="level" v-model="idChoosing" disabled>
        <label for="label">Label:</label>
        <textarea type="text" name="label" v-model="labelChoosing"></textarea>
      </div>
    </div>
  </div>
</template>
<script setup>
import { createApp } from 'vue'

import { Panel, PanelPosition, VueFlow, isNode, useVueFlow, MarkerType} from '@vue-flow/core'
import { Background } from '@vue-flow/background'
import { Controls } from '@vue-flow/controls'
import { MiniMap } from '@vue-flow/minimap'
import { ref, watch } from 'vue'

const isHidden = ref(false)
const nodes = ref([])
let total = ref(0)
let idChoosing = ref(0)
let labelChoosing = ref('')
let positionChoosing = ref({})
let levelChossing = ref(0)
let groupChoosing = ref(0)

watch(isHidden, () => {
  nodes.value.forEach((n) => (n.hidden = isHidden.value))
  edges.value.forEach((e) => (e.hidden = isHidden.value))
})

watch(labelChoosing, () => {
  nodes.value.forEach((item) => {
    if (item.id == idChoosing.value) {
      item.label = labelChoosing.value
    }
  })
})

const handleClickNode = (id) => {
  idChoosing.value = id
  nodes.value.forEach((item) => {
    if (item.id == id) {
      labelChoosing.value = item.label
      levelChossing.value = item.level
      groupChoosing.value = item.group
      positionChoosing.value.x = item.position.x
      positionChoosing.value.y = item.position.y
    }
  })
}

const onAdd = (type) => {
  const id = total.value + 1

  nodes.value.forEach((item) => {
    if (item.level > 1 && item.level > levelChossing.value) {
      item.level = item.level + 1
      item.position.y = item.position.y + 100
    }
  })

  if (nodes.value == []) {
    id = 1;
  }

  var bgColor = ''
  switch(type) {
  case 'input':
    bgColor = 'rgb(80 182 255 / 50%)'
    break;
  case 'decision':
  bgColor = 'rgb(0 175 21 / 50%)'
  break;
  case 'notice':
  bgColor = 'rgb(255 177 80 / 50%)'
    break;
  case 'output':
  bgColor = 'rgb(247 55 55 / 50%)'
    break;
}

  const newNode = {
    id: `${id}`,
    label: `Node ${id}`,
    draggable: false,
    level: levelChossing.value ? levelChossing.value + 1 : 1,
    group: id,
    events: {
      click: () => {handleClickNode(id)}
    },
    position: { x: positionChoosing.value.x ?? 300, y: positionChoosing.value.y ? positionChoosing.value.y + 100 : 100},
    type: type,
    style: { backgroundColor: bgColor},
  }

  if (id > 1) {
    const newEdge = {
      id: `el-${id}`,
      draggable: false,
      source: `${idChoosing.value}`,
      target: `${id}`,
      markerEnd: MarkerType.ArrowClosed,
      style: { stroke: '#41b6e6' },
    }
    nodes.value.push(newEdge)
  }

  var falseNode = {};
  nodes.value.forEach((item) => {
    if (item.level == levelChossing.value + 2) {
      falseNode = item
    }
  })

  nodes.value.forEach((item) => {
    if (item.source == `${idChoosing.value}` && item.target == falseNode.id) {
      item.source = `${id}`
    }
  })

  nodes.value.push(newNode)
  total.value++
}

const onAddCondition = () => {
  const id = idChoosing.value
  const idFalse = total.value + 1
  const idTrue = total.value + 2
  total.value = total.value + 2
  nodes.value.forEach((item) => {
    if (item.level > 1 && item.level > levelChossing.value) {
      item.level = item.level + 2
      item.position.y = item.position.y + 200
    }
  })
  const newNode = [
    {
      id: idFalse,
      label: `Node ${idFalse}`,
      draggable: false,
      in: id,
      out: ``,
      branch: 'false',
      group: id,
      level: levelChossing.value + 2,
      events: {
        click: () => {handleClickNode(idFalse)}
      },
      position: { x: positionChoosing.value.x, y: positionChoosing.value.y + 200},
      style: { backgroundColor: 'rgb(193 195 197 / 50%)'},
    },
    {
      id: idTrue,
      label: `Node ${idTrue}`,
      draggable: false,
      in: id,
      out: idFalse,
      branch: 'true',
      group: id,
      level: levelChossing.value + 1,
      events: {
        click: () => {handleClickNode(idTrue)}
      },
      position: { x: positionChoosing.value.x + 200, y: positionChoosing.value.y + 100 },
      style: { backgroundColor: 'rgb(193 195 197 / 50%)'},
    }
  ]
  newNode.forEach((item) => {
    
    nodes.value.push(item)
  })
  
    const newEdge = [
    {
      id: `el-${idFalse}`,
      draggable: false,
      type: 'smoothstep',
      label: 'false',
      type: 'special',
      source: `${id}`,
      target: `${idFalse}`,
      markerEnd: MarkerType.ArrowClosed
    },
    {
      id: `el-${idTrue}`,
      draggable: false,
      type: 'smoothstep',
      label: 'true',
      source: `${id}`,
      target: `${idTrue}`,
      markerEnd: MarkerType.ArrowClosed
    },
    {
      id: `el-${idTrue}-${idFalse}`,
      draggable: false,
      type: 'smoothstep',
      source: `${idTrue}`,
      target: `${idFalse}`,
      markerEnd: MarkerType.ArrowClosed
    },
  ]
  var falseNode = {};
  nodes.value.forEach((item) => {
    if (item.level == levelChossing.value + 3) {
      falseNode = item
    }
  })
  nodes.value.forEach((item) => {
    if (item.source == id && item.target == falseNode.id) {
      item.source = `${idFalse}`
    }
  })
  newEdge.forEach((item) => {
    nodes.value.push(item)
  })
}

const clearData = () => {
  nodes.value = []
  total.value = 0
  levelChossing.value = 0
  labelChoosing.value = ''
  positionChoosing.value = {}
}
const app = createApp({
  data() {
    return {
      
    }
  }
})

app.mount('#app')
</script>
