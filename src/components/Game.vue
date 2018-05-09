<template>
<div id="game">
	<ul class="player-list">
   <li class="player-info" v-for="user in users">
      <UserStatus :user="user"></UserStatus>
   </li> 
  </ul>
  <div class="system-info">
    <div class="system-message">{{systemMsg}}</div>
  	<div class="timer">{{currentTime}}</div>
    <div class="status">{{showStatus}}</div>
  	<div class="result" :style="resultStyle">{{showResult | toMessage}}</div>
  </div>
  <div class="roulette-box">
	 <Roulette :configs="configs" @reset="resetWheel"></Roulette> 
  </div>
  <ul class="card-shop">
    <li class="card" :style={background:card.color} v-for="card in cardShop">
      <div class="sensor" @click="addCard(card)">+</div>
      <div class="sensor" @click="cancelCard(card)">-</div>
    </li>
  </ul>
  <form class="join" v-if="userListEmpty">
    <p>請輸入名稱：</p>
    <input v-model="userName" name="name" type="text">
    <input type="submit" value="Enter" class="enter" @click="joinGame($event)">
  </form>
	<button v-if="isDebugging" @click="rotateWheel">Start</button>
</div>
</template>

<script>
import Roulette from '@/components/Roulette';
import UserStatus from '@/components/UserStatus';

export default {
  name: 'Game',
  mounted() {
    // Decide the roulette size width when mounted
    this.adjustRouletteWidth();
  	// Mock the game system
  	setInterval(this.game, 1000);
  },
  data () {
    return {
      // Debug Mode
      isDebugging: false,
      // input player name
      userName: null,
      // System message
      systemMsg: null,
      // mocking user data
      users: [],
      // card shop
      cardShop: [
        {color: 'dodgerblue', price: 1},
        {color: 'lightgreen', price: 1},
        {color: 'crimson', price: 1},
        {color: 'yellow', price: 1},
        {color: 'violet', price: 1},
        {color: 'gold', price: 1},
        {color: 'silver', price: 1},
      ],
      // card mark when mouse enter
      cardMark: null,
    	// rotate only if ready
    	ready: true,
      // can buy cards
      canBuy: true,
      // Timer Limit
      maxTime: 59,
    	// Timer
    	currentTime: 59,
      // game stage
      stage: [
        {
          limit: 25,
          text: '請玩家購買卡片'
        },
        {
          limit: 20,
          text: '輪盤即將開轉...'
        },
        {
          limit: 10,
          text: '輪盤轉動中...'
        },
        {
          limit: 0,
          text: '開獎！'
        }
      ],
    	// Result
    	result: null,
      // result switch
      resultReady: false,
    	// Roulette Configs
    	configs: {
    		// n-side polygon
    		side: 54,
    		// size of the polygon
    		radius: 300,
        // size of the roulette container
        boxSize: 2.1,
    		// rounds per rotation
    		round: 20,
    		// control rotate time
    		rotateTime: 10,
    		// current rotation state
    		currentRotate: 0,
    		// card types
    		cardList: [{color:"silver",money:52,total:1},{color:"dodgerblue",money:2,total:2},{color:"lightgreen",money:4,total:13},{color:"dodgerblue",money:2,total:2},{color:"crimson",money:7,total:7},{color:"lightgreen",money:4,total:13},{color:"dodgerblue",money:2,total:2},{color:"yellow",money:13,total:4},{color:"dodgerblue",money:2,total:2},{color:"crimson",money:7,total:7},{color:"dodgerblue",money:2,total:2},{color:"lightgreen",money:4,total:13},{color:"dodgerblue",money:2,total:2},{color:"violet",money:26,total:2},{color:"dodgerblue",money:2,total:2},{color:"lightgreen",money:4,total:13},{color:"dodgerblue",money:2,total:2},{color:"crimson",money:7,total:7},{color:"dodgerblue",money:2,total:2},{color:"lightgreen",money:4,total:13},{color:"dodgerblue",money:2,total:2},{color:"yellow",money:13,total:4},{color:"dodgerblue",money:2,total:2},{color:"crimson",money:7,total:7},{color:"dodgerblue",money:2,total:2},{color:"lightgreen",money:4,total:13},{color:"dodgerblue",money:2,total:2},{color:"gold",money:52,total:1},{color:"lightgreen",money:4,total:13},{color:"dodgerblue",money:2,total:2},{color:"lightgreen",money:4,total:13},{color:"dodgerblue",money:2,total:2},{color:"lightgreen",money:4,total:13},{color:"dodgerblue",money:2,total:1},{color:"yellow",money:13,total:4},{color:"dodgerblue",money:2,total:2},{color:"crimson",money:7,total:7},{color:"dodgerblue",money:2,total:2},{color:"lightgreen",money:4,total:13},{color:"dodgerblue",money:2,total:2},{color:"violet",money:26,total:2},{color:"dodgerblue",money:2,total:2},{color:"lightgreen",money:4,total:13},{color:"dodgerblue",money:2,total:2},{color:"crimson",money:7,total:7},{color:"dodgerblue",money:2,total:2},{color:"yellow",money:13,total:4},{color:"dodgerblue",money:2,total:2},{color:"lightgreen",money:4,total:13},{color:"dodgerblue",money:2,total:2},{color:"crimson",money:7,total:7},{color:"dodgerblue",money:2,total:2},{color:"lightgreen",money:4,total:13},{color:"dodgerblue",money:2,total:2}],
    	}
    }
  },
  computed: {
    userListEmpty() {
      return this.users.length === 0;
    },
    showStatus() {
      switch(true) {
        case (this.currentTime > this.stage[0].limit):
          return this.stage[0].text;
          break;
        case (this.currentTime > this.stage[1].limit):
          return `${this.stage[1].text} 還剩 ${this.currentTime - this.stage[1].limit} 秒`;
          break;
        case (this.currentTime > this.stage[2].limit):
          return this.stage[2].text;
          break;
        case (this.currentTime >= this.stage[3].limit):
          return this.stage[3].text;
          break;
        default:
          return `請重新整理頁面...`;
      }
    },

  	showResult() {
  		if(this.result && this.resultReady) {
  			return this.result;
  		}
  	},
    resultStyle() {
      if(this.result && this.resultReady) {
        return {
          color: this.result.color
        };
      }
    },
  },
  filters: {
  	toMessage(result) {
      let colorText = {
        dodgerblue: '藍卡',
        lightgreen: '綠卡',
        crimson: '紅卡',
        yellow: '黃卡',
        violet: '紫卡',
        gold: '金卡',
        silver: '鑽卡'
      }
  		if(result) return `恭喜 ${colorText[result.color]} 持有人獲得每張卡片 ${result.money} 元！`;
  	}
  },
  methods: {
    joinGame(event) {
      event.preventDefault();
      if(!this.userName) {
        alert('名字和人生一樣，不能留白！');
        return;
      }
      // create and add user
      this.users.push({
        name: this.userName.trim(),
        money: 50,
        holdCards: {
          dodgerblue: 0,
          lightgreen: 0,
          crimson: 0,
          yellow: 0,
          violet: 0,
          gold: 0,
          silver: 0
        }
      });
      // refresh input
      this.userName = null;
    },
    adjustRouletteWidth() {
      this.configs.radius = innerWidth / Math.sqrt(2) / 2;
    },
    // mock the main game
  	game() {
  		// update Timer in every second
  		if(this.currentTime > 0) {
  			this.currentTime --;
  		} else {
  			this.currentTime += this.maxTime;
  		}

      // user can't buy cards now
      if(this.currentTime === this.stage[0].limit) {
        this.canBuy = false;
      }

  		// start to rotate wheel when reaching 20s 
  		if(this.currentTime === this.stage[1].limit) {
  			this.rotateWheel();
  		}

      // show result when reaching 10s
      if(this.currentTime === this.stage[2].limit) {
        this.resultReady = true;
        // settle accounts
        this.settleAccounts();
      }

      // hide result when starting next round
      if(this.currentTime === this.maxTime) {
        this.resultReady = false;
        this.canBuy = true;
      }

  	},
    addCard(card) {
      if(!this.canBuy) {
        alert('Times up!');
        return;
      }

      if(this.users[0].money >= card.price) {
        this.users[0].money -= card.price;
        this.users[0].holdCards[card.color]++;          
      } else {
        alert('You have no money left!');
      }
    },
    cancelCard(card) {
      if(!this.canBuy) {
        alert('Times up!');
        return;
      }

      if(this.users[0].holdCards[card.color] > 0) {
        this.users[0].money += card.price;
        this.users[0].holdCards[card.color]--; 
      }
    },
  	rotateWheel() {
  		// set to ready
  		this.configs.ready = true;
  		// mock result target
  		let target = Math.floor(Math.random() * this.configs.side);
  		// calculate rotate result (in degree)
  		this.configs.currentRotate = -(this.configs.round + target / this.configs.side) * 360;
  		// update result data
  		this.result = this.configs.cardList[target];
      console.log(this.result)
  	},
  	resetWheel() {
  		this.configs.ready = false;
		  this.configs.currentRotate %= 360;
  	},
    settleAccounts() {
      // only the card holders can get the money
      this.users = this.users.map(user => {
        user.money += this.result.money * user.holdCards[this.result.color];
        console.log(this.result.money, user.holdCards[this.result.color], user.money)
        for(let card in user.holdCards) {
          user.holdCards[card] = 0;
        }
        return user;
      })
    },
  },
  components: {
  	Roulette,
    UserStatus
  }
}
</script>

<style lang="scss" scoped>
.player-info {
  border: 1px solid gray;
}
.card-shop {
  border: 1px solid gray;
  width: 100%;
  height: 80px;
  display: flex;
  justify-content: space-around;
  align-items: center;
  list-style-type: none;
  padding-left: 0;
  position: fixed;
  bottom: 0;
  margin: 0;
  .card {
    width: 12%;
    height: 60px;
    .sensor {
      height: 50%;
      width: 100%;
      display: flex;
      align-items: center;
      justify-content: center;
    }
  }
}
.roulette-box {
  padding: 2rem 0;
  display: flex;
  justify-content: center;
  position: relative;
  &:after {
    content: '';
    position: absolute;
    top: 2%;
    left: 50%;
    width: 0;
    height: 0;
    transform: translate(-50%);
    border-color: khaki transparent transparent;
    border-style: solid;
    border-width: 40px 10px;

  }
}

.system-info {
  text-align: center;
  .timer {
    font-size: 1.2rem;
    font-weight: bolder;
  }
  .result {
    min-height: 2rem;
    background: #222;
  }
}

.player-list {
  list-style-type: none;
  padding-left: 0;
  margin: 0;
}

.join {
  border: 1px solid black;
  background: #EEE;
  text-align: center;
  padding: 0.5rem;
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  * {
    margin: 0;
    margin-bottom: 0.5rem;
  }
  input {
    border: none;
  }
}
</style>