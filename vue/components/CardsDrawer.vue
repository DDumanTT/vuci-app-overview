<template>
  <a-drawer
    handle="drawerHandle"
    @close="$emit('close')"
    width="350"
    title="OVERVIEW SETTINGS"
    :closable="false"
    ref="drawer"
    :visible="visible"
  >
    <div class="box" @click="$emit('close')">
      <a-icon class="icon" type="setting" />
    </div>
    <a-checkbox-group
      v-model="checked"
      name="cardvisibility"
      :options="options"
      :style="{'display': 'grid', 'gap': '0.3em'}"
      @change="$emit('update-visible', $event)"
    />
  </a-drawer>
</template>

<script>
export default {
  props: ['visible', 'cards', 'visibleCards'],
  data () {
    return {
      checked: this.cards.filter(c => this.visibleCards.includes(c.name)).map(c => c.name)
    }
  },
  computed: {
    options () {
      return this.cards.map((c) => ({
        label: c.name.toUpperCase(),
        value: c.name
      }))
    }
  }
}
</script>

<style scoped>
.box {
  position: fixed;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 5px;
  height: 60px;
  width: 40px;
  background-color: #2d8cf0;
  top: 80px;
  right: 350px;
  border-top-left-radius: 0.8em;
  border-bottom-left-radius: 0.8em;
}

.icon {
  font-size: 1.4em;
  color: white;
}
</style>
