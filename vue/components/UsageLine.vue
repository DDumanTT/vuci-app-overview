<template>
  <div class="line">
    <span class="label" :style="{color}">{{ label }}: ({{ percent }}%)</span>
    <a-progress :percent="percent" :stroke-color="color" type="line" :show-info="false">
    </a-progress>
  </div>
</template>

<script>
export default {
  props: {
    percent: Number,
    label: String
  },
  computed: {
    color () {
      if (this.percent < 60) { return '#2d8cf0' }
      if (this.percent < 80) { return '#ff9900' }
      return '#ed4014'
    }
  }
}
</script>

<style scoped>
  .label {
    position: relative;
    top: 8px;
  }
</style>
