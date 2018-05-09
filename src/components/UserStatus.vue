<template>
<div class="user-status">
	<div class="name">{{userName}}</div>
	<div class="money">{{userMoney}}</div>
	<ul class="hold-cards">
		<li class="card" :style="{background: card.color}" v-for="card in holdCards">{{card.total}}</li>
	</ul>
</div>
</template>

<script>
export default {

  name: 'UserStatus',
  props:['user'],
  data () {
    return {

    }
  },
  computed: {
  	userName() {
  		return this.user.name;
  	},

  	userMoney() {
  		return this.user.money;
  	},

  	holdCards() {
  		let holdCardsArr = [];
  		for(let card in this.user.holdCards) {
  			let newCard = {
  				color: card,
  				total: this.user.holdCards[card]
  			};
  			if(newCard.total > 0) {
  				holdCardsArr.push(newCard);
  			}
  		}
  		return holdCardsArr;
  	},
  }
}
</script>

<style lang="scss" scoped>
.user-status {
	width: 100%;
	display: flex;
	text-align: center;
	.name {
		min-width: 120px;
		border-right: 1px solid gray;
	}
	.money {
		min-width: 60px;
		border-right: 1px solid gray;
	}
}

.hold-cards {
	padding-left: 0;
	list-style-type: none;
	display: flex;
	flex-grow: 1;
	flex-wrap: wrap;
	justify-content: space-around;
	.card {
		min-width: 14%;
	}
}
</style>