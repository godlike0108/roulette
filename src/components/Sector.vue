<template>
  <div class="circle-sector" :style="sectorClass"></div>
</template>

<script>
export default {
  name: '',
  data () {
    return {
      // triangle size
      halfEdge: this.roulette.sectorHalfEdge,
      height: this.roulette.sectorHeight,
    }
  },
  computed: {
    sectorClass() {
      return {
        borderTopColor: this.settings.color,
        borderTopWidth: `${this.height}px`,
        borderLeftWidth: `${this.halfEdge}px`,
        borderRightWidth: `${this.halfEdge}px`,
        borderBottomWidth: `0px`,
        transform: `rotate(${this.settings.arc * 360 / (Math.PI * 2)}deg)`,
        left: `calc(50% - ${this.halfEdge}px)`,
        top: `calc(50% - ${this.height}px`,
      }
    }
  },
  props:['settings', 'roulette']
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
.circle-sector {
  border-style: solid;
  border-color: transparent;
  width: 0;
  height: 0;
  position: absolute;
  transform-origin: center bottom;
}
</style>
