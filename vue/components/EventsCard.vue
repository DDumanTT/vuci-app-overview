<template>
  <div>
    <h2 :style="{ 'margin-bottom': 0 }">RECENT EVENTS</h2>
    <hr />
    <div v-for="(log, index) in latestLogs" :key="index">
      <h3>{{ formatDate(log.datetime) }}</h3>
      <span>{{ log.msg }}</span>
      <hr v-if="index < eventsCount - 1" class="light-hr" />
    </div>
  </div>
</template>

<script>

export default {
  components: { },
  data () {
    return {
      syslog: [],
      loading: true,
      eventsCount: 4
    }
  },
  methods: {
    formatDate (time) {
      const d = new Date(time)
      const year = d.getFullYear().toString().padStart(2, 0)
      const month = (d.getMonth() + 1).toString().padStart(2, 0)
      const date = d.getDate().toString().padStart(2, 0)
      const hour = d.getHours()
      const min = d.getMinutes()
      const sec = d.getSeconds()
      return `${year}-${month}-${date} %02d:%02d:%02d`.format(hour, min, sec)
    },
    async getSystemLog () {
      const log = await this.$rpc.call('system', 'syslog')
      this.syslog = log.log
      this.loading = false
    }
  },
  computed: {
    latestLogs () {
      return this.syslog.slice(-this.eventsCount).reverse()
    }
  },
  created () {
    this.getSystemLog()
  }
}
</script>

<style scoped>
  .light-hr {
    border-top: none;
  }
  h3 {
    font-size: 1em;
  }
  span {
    font-size: 0.9em;
  }
</style>
