<template>
  <div>
    <a-space :style="{ width: '100%', 'justify-content': 'space-between' }">
      <h2 :style="{ 'margin-bottom': 0 }">SYSTEM</h2>
      <usage-line label="CPU" :percent="cpuUsage"></usage-line>
    </a-space>
    <hr />
    <h3>ROUTER UPTIME</h3>
    <span>{{ "%t".format(system.uptime) }}</span>
    <hr class="light-hr" />
    <h3>LOCAL DEVICE TIME</h3>
    <span>{{ localTime }}</span>
    <hr class="light-hr" />
    <h3>MEMORY USAGE</h3>
    <a-space :style="{ width: '100%' }">
      <div>
        <usage-line label="RAM" :percent="ramUsage"></usage-line>
      </div>
      <div>
        <usage-line label="FLASH" :percent="flashUsage"></usage-line>
      </div>
    </a-space>
    <hr class="light-hr" />
    <h3>FIRMWARE VERSION</h3>
    <span>{{ system.release && system.release.revision }}</span>
  </div>
</template>

<script>
import UsageLine from './UsageLine.vue'

export default {
  components: { UsageLine },
  data () {
    return {
      system: {},
      loading: true,
      uptime: 0,
      ramUsage: 0,
      flashUsage: 0,
      lastCPUTime: null,
      cpuUsage: 0
    }
  },
  timers: {
    update: { time: 1000, autostart: true, repeat: true, immediate: true },
    updateCpuUsage: {
      time: 1000,
      autostart: true,
      immediate: true,
      repeat: true
    }
  },
  methods: {
    async getSystemInfo () {
      this.system = await this.$system.getInfo()
      this.loading = false
    },
    async update () {
      this.system = await this.$system.getInfo()
      this.ramUsage = Math.floor(
        ((this.system.memory.total - this.system.memory.free) /
          this.system.memory.total) *
          100
      )
      const { root } = await this.$system.getDiskInfo()
      this.flashUsage = Math.floor((root.used / root.total) * 100)
    },
    updateCpuUsage () {
      this.$rpc.call('system', 'cpu_time').then((times) => {
        if (!this.lastCPUTime) {
          this.cpuUsage = 0
          this.lastCPUTime = times
          return
        }

        const idle1 = this.lastCPUTime[3]
        const idle2 = times[3]

        let total1 = 0
        let total2 = 0

        this.lastCPUTime.forEach((t) => {
          total1 += t
        })

        times.forEach((t) => {
          total2 += t
        })

        this.cpuUsage = Math.floor(
          ((total2 - total1 - (idle2 - idle1)) / (total2 - total1)) * 100
        )
        this.lastCPUTime = times
      })
    }
  },
  computed: {
    localTime () {
      const d = new Date(this.system.localtime * 1000)
      const year = d.getFullYear().toString().padStart(2, 0)
      const month = (d.getMonth() + 1).toString().padStart(2, 0)
      const date = d.getDate().toString().padStart(2, 0)
      const hour = d.getHours()
      const min = d.getMinutes()
      const sec = d.getSeconds()
      return `${year}-${month}-${date} %02d:%02d:%02d`.format(hour, min, sec)
    }
  },
  created () {
    this.getSystemInfo()
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
