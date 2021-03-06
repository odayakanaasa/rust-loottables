<template>
  <div class="gradients">
    <div ref="items" class="items" :class="{ 'moving' : scroll.enabled }">
      <transition-group name="list" class="transition">
        <div class="item" v-for="loot in crate.loots" :key="loot.id" @dblclick="openRustLabs(loot)">
          <div class="info">
            <span v-if="loot.amount" class="amount">{{ loot.amount }}</span>
            <span v-if="loot.percentage" class="percentage" :class="{ 'green' : loot.percentage >= maxPercentage }">{{ loot.percentage }}%</span>
          </div>
          <div class="image">
            <div class="icon" :style="{ backgroundImage: `url(https://rustlabs.com/img/items180/${loot.dataId}.png)` }"></div>
            <div v-if="loot.blueprint" class="blueprint"></div>
          </div>
          <div class="title">{{ loot.name }}</div>
        </div>
      </transition-group>
    </div>
    <div v-show="scroll.start" class="gradient-left"></div>
    <div v-show="!scroll.end" class="gradient-right"></div>
  </div>
</template>

<script>
export default {
  name: 'LootList',
  props: ['crate'],
  data () {
    return {
      scroll: {
        enabled: false,
        clientX: 0,
        scrollLeft: 0,
        start: 0,
        end: false
      }
    }
  },
  computed: {
    maxPercentage () {
      const percentages = this.crate.loots.map((item) => item.percentage)
      return Math.max(...percentages)
    }
  },
  mounted () {
    this.$refs.items.addEventListener('mousedown', (event) => {
      this.scroll.enabled = true
      this.scroll.scrollLeft = this.$refs.items.scrollLeft
      this.scroll.start = this.$refs.items.scrollLeft
      this.scroll.clientX = event.pageX
    })
    this.$refs.items.addEventListener('mouseup', () => {
      this.scroll.enabled = false
    })
    this.$refs.items.addEventListener('mouseleave', () => {
      this.scroll.enabled = false
    })
    this.$refs.items.addEventListener('mousemove', (event) => {
      event.preventDefault()
      if (this.scroll.enabled) {
        this.$refs.items.scrollLeft = this.scroll.scrollLeft + this.scroll.clientX - event.pageX
        this.scroll.start = this.$refs.items.scrollLeft
        this.scroll.end = this.$refs.items.scrollWidth - this.$refs.items.scrollLeft === this.$refs.items.clientWidth
      }
    })
  },
  methods: {
    openRustLabs (loot) {
      window.open(`https://rustlabs.com/item/${this.slugify(loot.name)}`, '_blank')
    },
    slugify (text) {
      return text.toString().toLowerCase().trim()
        .replace(/[\s_-]+/g, '-') // swap any length of whitespace, underscore, hyphen characters with a single -
        .replace(/^-+|-+$/g, '') // remove leading, trailing -
        .replace('-blueprint', '') // remove blueprint from name
    }
  }
}
</script>

<style lang="scss">
@import 'assets/variables.scss';

.gradients {
  position: relative;
  .gradient-right {
    position: absolute;
    z-index: 5;
    top: 0px;
    right: 0px;
    width: 100px;
    height: 100%;
    background: linear-gradient(to left, $primaryBackground 0%,rgba(125,185,232,0) 70%);
  }
  .gradient-left {
    position: absolute;
    z-index: 5;
    top: 0px;
    left: 0px;
    width: 100px;
    height: 100%;
    background: linear-gradient(to right, $primaryBackground 0%,rgba(125,185,232,0) 70%);
  }
}

.items {
  user-select: none;
  overflow-x: auto;
  overflow: -moz-scrollbars-none; // FFvvv
  -ms-overflow-style: none;  // IE 10+
  &::-webkit-scrollbar {
    display: none;  // Safari and Chrome
  }
  &.moving {
    cursor: w-resize;
  }
  .transition {
    display: flex;
  }
}

.item {
  background: $secondaryBackground;
  height: 128px;
  padding: 8px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  position: relative;
  margin-right: 2px;
  flex: 0 0 120px;
  .info {
    position: absolute;
    top: 6px;
    right: 6px;
    display: flex;
    justify-content: flex-end;
    align-items: center;
    opacity: .4;
    span {
      background: $gray;
      color: $white;
      font-size: 10px;
      border-radius: 3px;
      padding: 2px 4px;
      margin-left: 2px;
      &.percentage {
        background: darken($secondary, 10%);
        &.green {
          background: darken($primary, 10%);
        }
      }
    }
  }
  &:hover {
    background: lighten($secondaryBackground, 2%);
    .info {
      opacity: 1;
    }
  }
  .title {
    text-align: center;
    font-size: 14px;
  }
  .image {
    position: relative;
    margin: 8px;
    width: 60px;
    height: 60px;
    .icon {
      position: relative;
      background-size: contain;
      background-position: center center;
      background-repeat: no-repeat;
      width: 100%;
      height: 100%;
      z-index: 1;
    }
    .blueprint {
      position: absolute;
      background: url(/blueprint.png) center center no-repeat;
      background-size: contain;
      top: 0;
      width: 100%;
      height: 100%;
      z-index: 0;
    }
  }
}

.list-enter-active, .list-leave-active {
  transition: all .3s;
}
.list-enter, .list-leave-to {
  opacity: 0;
}
</style>
