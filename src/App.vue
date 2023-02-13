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
      <VueFlow 
        v-model="nodes"
        :zoom-on-double-click="false"
        :zoom-on-scroll="false"
        :prevent-scrolling="false"
        :pan-on-drag="false"
        :pan-on-scroll="true"
        auto-connect
        >
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
        <label for="level">Level-Y:</label>
        <input type="text" name="level" v-model="levelChossing" disabled>
        <label for="levelX">Level-X:</label>
        <input type="text" name="levelX" v-model="levelXChossing" disabled>
        <label for="label">Label:</label>
        <textarea type="text" name="label" v-model="labelChoosing"></textarea>
        <div class="group-button">
          <button type="button" class="clear-button" @click="clearNode()">Delete</button>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup>
import { createApp } from 'vue'

import { Panel, PanelPosition, VueFlow, isNode, Position, MarkerType} from '@vue-flow/core'
import { Background } from '@vue-flow/background'
import { Controls } from '@vue-flow/controls'
import { MiniMap } from '@vue-flow/minimap'
import { ref, watch } from 'vue'
import ColorSelectorNode from './components/custom.vue'

const nodes = ref([])
const total = ref(0)
const idChoosing = ref(0)
const labelChoosing = ref('')
const positionChoosing = ref({})
const levelChossing = ref(0)
const levelXChossing = ref(0)
const groupChoosing = ref(0)

watch(labelChoosing, () => {
  nodes.value.forEach((item) => {
    if (item.id == idChoosing.value) {
      item.label = labelChoosing.value
    }
  })
})

function clearNode() {
  var buttonDeletes = document.getElementsByClassName('delete-button')
  var index = 0
  while (buttonDeletes.length) {
    buttonDeletes[index].remove()
  }

  var quantity = 0;
  var i = nodes.value.length
  var j = nodes.value.length
  var n = nodes.value.length
  var levelLow = 0;
  nodes.value.forEach((item) => {
    if (item.in == idChoosing.value && item.levelX == levelXChossing.value) {
      levelLow = item.level
    }
  })
  if (!levelLow) {
    var before = 0
    var after = 0
    nodes.value.forEach((item, key) => {
      for (var i = 1; i<= total.value; i++) {
        var e = document.getElementById(i)
        if (e && e.id == levelChossing.value + 1) {
          e.remove()
        }
      }
      if (item.level == levelChossing.value + 1) {
        after = item.id
      } 
      if (item.level == levelChossing.value - 1) {
        before = item.id
      }
    })
    while (n--) {
        if (nodes.value[n]?.source == before && nodes.value[n].target == idChoosing.value) { 
          nodes.value.splice(n, 1)
        } 
        if (nodes.value[n]?.source == idChoosing.value && nodes.value[n].target == after) { 
          nodes.value.splice(n, 1)
        } 
    }

    const newEdge = 
      {
        id: `el-${before}-${after}`,
        draggable: false,
        type: 'smoothstep',
        type: 'special',
        source: `${before}`,
        target: `${after}`,
        markerEnd: MarkerType.ArrowClosed
      }
      nodes.value.push(newEdge)
      while (j--) {
        if (nodes.value[j]?.id == idChoosing.value) { 
          nodes.value.splice(j, 1)
        } 
    }
    nodes.value.forEach((item) => {
      if (item.level > levelChossing.value) {
        item.position.y = item.position.y - 100
        item.level = item.level - 1
      }
    })
    return 0;
  }
  while (i--) {
      if (nodes.value[i]?.level > levelChossing.value && nodes.value[i]?.level < levelLow|| nodes.value[i]?.id == idChoosing.value) { 
        quantity++
        nodes.value.splice(i, 1)
      } 
  }
  nodes.value.forEach((item, key) => {
    if (item.level == levelChossing.value + quantity) {
      quantity++
      nodes.value.splice(key, 1)
    }
  })
  nodes.value.forEach((item) => {
    if (item.level >= levelChossing.value + quantity) {
      item.position.y = item.position.y - 100 * quantity
    }
  })
}

const handleClickNode = (id) => {
  idChoosing.value = id
  nodes.value.forEach((item) => {
    if (item.id == id) {
      labelChoosing.value = item.label
      levelChossing.value = item.level
      levelXChossing.value = item.levelX
      groupChoosing.value = item.group
      positionChoosing.value.x = item.position.x
      positionChoosing.value.y = item.position.y
    }
  })

  //show button delete
  var buttonDeletes = document.getElementsByClassName('delete-button')
  var index = 0
  while (buttonDeletes.length) {
    buttonDeletes[index].remove()
  }

  var panel = document.getElementsByClassName("vue-flow__nodes")[0]
  var thisNode = event.currentTarget
  var thisNodePosititon = thisNode.getBoundingClientRect()
  var header = document.getElementsByClassName('header')[0]
  var menu = document.getElementsByClassName('container-items')[0]
  var deleteButton = document.createElement('div')

  deleteButton.innerHTML = '<i class="fa fa-times-circle-o" aria-hidden="true"></i>'
  deleteButton.classList.add('delete-button')
  deleteButton.style.position = 'absolute'
  deleteButton.style.top = (thisNodePosititon.top - header.offsetHeight - 20) + 'px'
  deleteButton.style.left = (thisNodePosititon.left - menu.offsetWidth + 500) + 'px'
  deleteButton.onclick = function() {clearNode()}
  panel.append(deleteButton)
}

function handleCollapse (id,levelX, level) {console.log(id,levelX, level)
  var quantity = 0;
  var levelLow = 0;
  var isCollapse = false
  nodes.value.forEach((item) => {
    if (item.in == id && item.levelX == levelX) {
      levelLow = item.level
    }
    if (item.id == id) {
      if (item.collapse) {
        item.collapse = false
      } else {
        isCollapse = true
        item.collapse = true
      }
    }
  })
  nodes.value.forEach((item) => {
    if (item.level > level && item.level < levelLow) {
      quantity++
      var numberOfNode = document.getElementById(item.level)
      var collapse = document.getElementById(`collapse-${item.level}`)
      if (item.hidden) {
        item.hidden = false
        numberOfNode.hidden = false
        if (collapse) {
          collapse.hidden = false
        }
      } else {
        item.hidden = true
        numberOfNode.hidden = true
        if (collapse) {
          collapse.hidden = true
        }
      }
    }
  })
  nodes.value.forEach((item) => {
    if (item.level > level + quantity) {
      var numberOfNode = document.getElementById(item.level)

      if (isCollapse) {
        numberOfNode.style.top = `calc(${numberOfNode.style.top} - ${100 * quantity}px)`
        item.position.y = item.position.y - 100 * quantity
      } else {
        numberOfNode.style.top = `calc(${numberOfNode.style.top} + ${100 * quantity}px)`
        item.position.y = item.position.y + 100 * quantity
      }
    }
  })
}

const onAdd = (type) => {
  const id = total.value + 1

  nodes.value.forEach((item) => {
    if (item.level > 1 && item.level > levelChossing.value) {
      var numberOfNode = document.getElementById(item.level)
      var collapse = document.getElementById(`collapse-${item.level}`)
      numberOfNode.innerHTML = item.level + 1
      numberOfNode.id = item.level + 1
      numberOfNode.style.top = `calc(${numberOfNode.style.top} + ${100}px)`
      if (collapse) {
        collapse.style.top = `calc(${collapse.style.top} + ${100}px)`
        collapse.id = `collapse-${item.level + 1}`
      }
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
    levelX: levelXChossing.value ? levelXChossing.value : 1,
    group: id,
    collapse: false,
    events: {
      click: () => {handleClickNode(id)}
    },
    position: { x: positionChoosing.value.x ?? 100, y: positionChoosing.value.y ? positionChoosing.value.y + 100 : 100},
    type: type,
    style: { backgroundColor: bgColor, borderRadius: '5px'},
    sourceHandle:'left'
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

  const panel = document.getElementsByClassName("vue-flow__nodes")[0]
  const number = document.createElement("div")
  const text = document.createTextNode(levelChossing.value ? levelChossing.value + 1 : 1);
  number.style.position = 'absolute'
  number.style.left = '70px'
  number.id = levelChossing.value ? levelChossing.value + 1 : 1
  number.style.top = positionChoosing.value.y ? `${positionChoosing.value.y + 109}px` : '109px'
  number.appendChild(text)
  panel.append(number)

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
  const idCenter = total.value + 1
  const idFalse = total.value + 2
  const idTrue = total.value + 3
  total.value = total.value + 3
  const lv = levelChossing.value
  const lvX = levelXChossing.value
  nodes.value.forEach((item) => {
    if (item.level > 1 && item.level > levelChossing.value) {
      var numberOfNode = document.getElementById(item.level)
      numberOfNode.id = item.level + 3
      numberOfNode.innerHTML = item.level + 3
      numberOfNode.style.top = `calc(${numberOfNode.style.top} + ${300}px)`
      item.level = item.level + 3
      item.position.y = item.position.y + 300
    }
  })
  const newNode = [
    {
      id: idCenter,
      label: `Node ${idCenter}`,
      draggable: false,
      in: id,
      out: ``,
      branch: '',
      group: id,
      collapse: false,
      level: levelChossing.value + 1,
      levelX: levelXChossing,
      events: {
        click: () => {handleClickNode(idCenter)}
      },
      position: { x: positionChoosing.value.x, y: positionChoosing.value.y + 100},
      style: { backgroundColor: 'rgb(193 195 197 / 50%)'},
    },
    {
      id: idFalse,
      label: `Node ${idFalse}`,
      draggable: false,
      in: idCenter,
      out: ``,
      branch: 'false',
      group: idCenter,
      collapse: false,
      level: levelChossing.value + 3,
      levelX: levelXChossing,
      events: {
        click: () => {handleClickNode(idFalse)}
      },
      position: { x: positionChoosing.value.x, y: positionChoosing.value.y + 300},
      style: { backgroundColor: 'rgb(193 195 197 / 50%)'},
    },
    {
      id: idTrue,
      label: `Node ${idTrue}`,
      draggable: false,
      in: idCenter,
      out: idFalse,
      branch: 'true',
      group: idCenter,
      collapse: false,
      level: levelChossing.value + 2,
      levelX: levelXChossing.value + 1,
      events: {
        click: () => {handleClickNode(idTrue)}
      },
      position: { x: positionChoosing.value.x + 50, y: positionChoosing.value.y + 200 },
      style: { backgroundColor: 'rgb(193 195 197 / 50%)'},
    }
  ]
  newNode.forEach((item) => {
    nodes.value.push(item)
  })

  const panel = document.getElementsByClassName("vue-flow__nodes")[0]
  const collapse = document.createElement("div")
  collapse.classList.add("arrow-right")
  collapse.id = `collapse-${levelChossing.value + 1}`
  collapse.style.left = '85px'
  collapse.style.top = `${positionChoosing.value.y + 110}px`
  collapse.style.pointerEvents = 'all'
  collapse.onclick = function() {handleCollapse(idCenter, lvX, lv + 1)}
  panel.append(collapse)
  for (var i = 1; i <= 3; i++) {
    var number = document.createElement("div")
    var text = document.createTextNode(levelChossing.value + i);
    number.style.position = 'absolute'
    number.style.left = '70px'
    number.style.top = `${positionChoosing.value.y + (100 * i) + 10}px`
    number.setAttribute('id', levelChossing.value + i)
    number.appendChild(text)
    panel.append(number)
  }
  
    const newEdge = [
      {
      id: `el-${id}-${idCenter}`,
      draggable: false,
      type: 'smoothstep',
      type: 'special',
      source: `${id}`,
      target: `${idCenter}`,
      markerEnd: MarkerType.ArrowClosed
    },
    {
      id: `el-${idFalse}`,
      draggable: false,
      type: 'smoothstep',
      label: 'false',
      type: 'special',
      source: `${idCenter}`,
      target: `${idFalse}`,
      markerEnd: MarkerType.ArrowClosed
    },
    {
      id: `el-${idTrue}`,
      draggable: false,
      type: 'smoothstep',
      label: 'true',
      source: `${idCenter}`,
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
    if (item.level == levelChossing.value + 4) {
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
  for (var i = 1; i<= total.value; i++) {
    var e = document.getElementById(i)
    var collapse = document.getElementById(`collapse-${i}`)
    if (e) {
      e.remove()
    }
    if (collapse) {
      collapse.remove()
    }
  }
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
