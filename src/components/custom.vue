<script setup>
import { Handle, Position } from '@vue-flow/core'
import { computed } from 'vue'

const props = defineProps({
  data: {
    type: Object,
    required: true,
  },
})

const emit = defineEmits(['change', 'gradient'])

const onGradient = () => {
  emit('gradient')
}

const sourceHandleStyleA = computed(() => ({ filter: 'invert(100%)', top: '10px' }))

const sourceHandleStyleB = computed(() => ({

  filter: 'invert(100%)',
  bottom: '20px',
  top: 'auto',
}))
</script>

<template>
  <div>Select a color</div>

  <div
    style="display: flex; flex-direction: row; flex-wrap: wrap; justify-content: center; max-width: 90%; margin: auto; gap: 3px"
  >
    

    <button class="animated-bg-gradient" title="gradient" type="button"></button>
  </div>

  <Handle type="source" :position="Position.Right" :style="sourceHandleStyleA" />
</template>