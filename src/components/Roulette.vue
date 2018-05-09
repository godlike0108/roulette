<template>
	<div class="roulette" :style="rouletteClass" @transitionend="emitReset">
		<template v-for="item in sectorList">
			<Sector :settings="item" :roulette="roulette"></Sector>
		</template>
	</div>
</template>

<script>
import Sector from '@/components/Sector';
export default {

  name: 'Roulette',
  props:['configs'],
  data () {
  	return {
  	}
  },
  computed: {
  	roulette() {
  		return {
  			r: this.configs.radius,
  			size: this.configs.radius * Math.cos(Math.PI * 2 / (this.configs.side * 2)) * 2,
  			sectorHeight: this.configs.radius * Math.cos(Math.PI * 2 / (this.configs.side * 2)),
  			sectorHalfEdge: this.configs.radius * Math.sin(Math.PI * 2 / (this.configs.side * 2)),
  		}
  	},
  	sectorList() {
  		let list = this.configs.cardList;
  		return list.map((item, index) => {
  			item.arc = index * Math.PI * 2 /this.configs.side;
  			return item;
  		})
  	},
  	rouletteClass() {
  		let style = {
  			width: `${this.roulette.size}px`,
  			height: `${this.roulette.size}px`,
  			transform: `rotate(${this.configs.currentRotate}deg)`,
  		};

  		// no transition if not ready
  		if(this.configs.ready) {
  			style.transition = `transform ${this.configs.rotateTime}s`;
  		}

  		return style;
  	}
  },
  methods: {
  	emitReset() {
  		this.$emit('reset');
  	}
  },
  components: {
  	Sector,
  }
}
</script>

<style lang="scss" scoped>
.roulette {
	position: relative;
	border-radius: 50%;
	overflow: hidden;
}
</style>