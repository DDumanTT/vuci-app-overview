<template>
  <div>
    <h2>INTERFACE {{ iface.name.toUpperCase() }}</h2>
    <hr />
    <h3>PROTOCOL</h3>
    <span class="capital-first">{{ iface.status.proto }}</span>
    <hr class="light-hr" />
    <h3>IP ADDRESS</h3>
    <span>{{ iface.getIPv4Addrs("")[0] || "-" }}</span>
  </div>
</template>

<script>

export default {
  components: { },
  props: ['iface'],
  methods: {}
}
</script>

<style scoped>
  .light-hr {
    border-top: none;
  }
  .capital-first {
    text-transform: capitalize;
  }
  h3 {
    font-size: 1em;
  }
  span {
    font-size: 0.9em;
  }
  h2 {
    word-wrap: break-word;
  }
</style>
