<template>
  <div v-if="!loading">
    <transition-group tag="div" name="cards-list" class="container" >
      <draggable-card
        @reorder="handleReorder"
        v-for="card in filteredCards"
        :key="card.id"
        :card="card"
      />
    </transition-group>
    <cards-drawer
      :visible="drawerVisible"
      :cards="cards"
      :visibleCards="visible"
      @close="drawerVisible = !drawerVisible"
      @update-visible="updateVisible"
    />
  </div>
</template>

<script>
import DraggableCard from './components/DraggableCard.vue'
import CardsDrawer from './components/CardsDrawer.vue'

export default {
  components: { DraggableCard, CardsDrawer },
  data () {
    return {
      cards: [],
      order: [],
      visible: [],
      loading: true,
      drawerVisible: false
    }
  },
  methods: {
    handleReorder ({ from, to }) {
      const fromIndex = this.order.indexOf(from)
      const toIndex = this.order.indexOf(to)
      const name = this.order.splice(fromIndex, 1)[0]
      this.order.splice(toIndex, 0, name)

      this.updateOrder()
    },
    async updateOrder () {
      this.$uci.reset()
      await this.$uci.load('overview')
      this.$uci.set('overview', 'order', 'names', this.order)
      await this.$uci.save()
      await this.$uci.apply()
    },
    async updateVisible (checked) {
      this.$uci.reset()
      await this.$uci.load('overview')
      this.visible = checked
      this.$uci.set('overview', 'visible', 'names', checked)
      await this.$uci.save()
      await this.$uci.apply()
    },
    UciAddCard (component, name) {
      const sid = this.$uci.add('overview', 'card')
      this.$uci.set('overview', sid, 'component', component)
      this.$uci.set('overview', sid, 'name', name)
    },
    async deleteCards () {
      this.$uci.reset()
      await this.$uci.load('overview')
      this.$uci.sections('overview', 'card').forEach(c =>
        this.$uci.del('overview', c['.name'])
      )
      await this.$uci.save()
      await this.$uci.apply()
    },
    async initializeCards () {
      this.$uci.reset()
      await this.$uci.load('overview')
      this.UciAddCard('SystemCard', 'system')
      this.UciAddCard('EventsCard', 'recent system events')
      this.$network.getInterfaces().forEach((iface) => {
        this.UciAddCard('InterfaceCard', iface.name)
      })
    },
    async loadCards () {
      await this.deleteCards()
      await this.$network.load()
      await this.initializeCards()
      const cards = this.$uci.sections('overview', 'card')

      cards.forEach((c, i) => (c.id = i))

      let order = this.$uci.get('overview', 'order')
      if (!order) {
        this.$uci.add('overview', 'order', 'order')
        this.$uci.set(
          'overview',
          'order',
          'names',
          cards.map((c) => c.name)
        )
        order = this.$uci.get('overview', 'order')
      }

      cards.forEach(c => {
        if (!order.names.includes(c.name)) order.names.push(c.name)
      })

      let visible = this.$uci.get('overview', 'visible')
      if (!visible) {
        this.$uci.add('overview', 'visible', 'visible')
        this.$uci.set('overview', 'visible', 'names', cards.map(c => c.name))
        visible = this.$uci.get('overview', 'visible')
      }
      this.cards = cards
      this.order = order.names || []
      this.visible = visible.names || []

      await this.$uci.save()
      await this.$uci.apply()

      this.loading = false
    }
  },
  computed: {
    filteredCards () {
      const cards = []
      this.order.forEach((name) => {
        const card = this.cards.find((c) => c.name === name && this.visible.includes(name))
        if (!card) return
        cards.push({
          name: card.name,
          component: card.component,
          id: Number(card.id)
        })
      })

      cards.forEach((c) => {
        if (c.component === 'InterfaceCard') {
          const iface = this.$network.getInterfaces().find((i) => i.name === c.name)
          c.props = {
            iface
          }
        }
      })
      return cards
    }
  },
  created () {
    this.loadCards()
  }
}
</script>

<style scoped>
.container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(350px, 1fr));
  grid-auto-rows: 1fr;
  grid-gap: 1rem;
}

.cards-list-item {
  transition: all 1s;
}

.cards-list-enter, .cards-list-leave-to{
  opacity: 0;
  transform: translateY(30px);
}
.cards-list-leave-active {
  position: absolute;
}
</style>
