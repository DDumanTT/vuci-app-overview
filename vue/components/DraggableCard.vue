<template>
  <a-card
    :style="{height: '100%'}"
    :draggable="true"
    :hoverable="true"
    @dragstart="dragStart"
    @drop="onDrop"
    @dragover.prevent
    @dragenter.prevent
    >
    <component :is="card.component" v-bind="card.props"></component>
  </a-card>
</template>

<script>
import SystemCard from './SystemCard.vue'
import EventsCard from './EventsCard.vue'
import InterfaceCard from './InterfaceCard.vue'

export default {
  components: { SystemCard, EventsCard, InterfaceCard },
  props: ['card'],
  methods: {
    dragStart (e) {
      e.dataTransfer.setData('name', this.card.name)
    },
    onDrop (e) {
      const name = e.dataTransfer.getData('name')
      this.$emit('reorder', { from: name, to: this.card.name })
    }
  }
}
</script>
