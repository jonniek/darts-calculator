<template>
  <div id="shootout">
    <div class="head">
      <h2>{{ this.currentround }}/{{ this.roundLimit }}</h2>
      <div class="record">
        <div>{{ record[0] | totext }}</div>
        <div>{{ record[1] | totext }}</div>
        <div>{{ record[2] | totext }}</div>
      </div>
    </div>
    <div class="players">
      <div v-for="(player, index) in players" v-bind:key="player.name" :class="{ active: turn === index, player }">
        <h3>{{ player.name }}</h3>
        <span class="score">{{ player.score }}</span>
        <span class="multiplier">{{ player.multiplier }}x</span>
      </div>
    </div>
    <div v-if="playerchange" class="playerchange" @click="changeplayer">Player change</div>
    <div v-if="gameover" class="gameover" @click="init">Game over</div>
    <Dartboard v-on:hit="handleDartHit"/>
  </div>
</template>

<script>
import Dartboard from './components/Dartboard.vue'
import { setTimeout } from 'timers';

export default {
  name: 'shootout',
  components: {
    Dartboard
  },
  props: ['playercount'],
  data: function() {
    return {
      turn: 0,
      roundLimit: 2,
      currentround: 1,
      players: [],
      record: [],
      playerchange: false,
      gamover: false,
    }
  },
  created() {
    this.init()
  },
  filters: {
    totext(score) {
      if (score === undefined) {
        return ''
      }
      if (score === null) {
        return 'miss'
      }
      if ('sio'.includes(score[0])) {
        return 'single ' + score.substr(1)
      }
      if (score[0] === 'd') {
        return 'double ' + score.substr(1)
      }
      if (score[0] === 't') {
        return 'triple ' + score.substr(1)
      }
      return score
    }
  },
  methods: {
    init() {
      this.gameover = false
      this.record = []
      this.currentround = 1
      this.turn = 0
      const players = []
      for (let i = 0; i < this.playercount; i++) {
        players.push({ name: "player " + (i+1), score: 0, throws: [], multiplier: 1, })
      }
      this.players = players
      setTimeout(() => {
        this.highlight(this.players[this.turn].throws)
      }, 10)
    },
    highlight(throws) {
      document.querySelectorAll('.bull,.a20,.a19,.a18,.a17,.a16,.a15,.a14,.a13,.a12,.a11,.a10,.a9,.a8,.a7,.a6,.a5,.a4,.a3,.a2,.a1')
        .forEach(item => item.style.fill = "#444")
      const allslices = ['bull','a20','a19','a18','a17','a16','a15','a14','a13','a12','a11','a10','a9','a8','a7','a6','a5','a4','a3','a2','a1']
      const validtargets = allslices.filter(slice => !throws.includes(slice))
      if (validtargets.length) {
        const query = validtargets.map(t => '.'+t).join(",")
        document.querySelectorAll(query).forEach(t => t.style.fill = "inherit")
      }
    },
    handleDartHit(hit) {
      const { score, target } = this.calculateValue(hit)
      const player = this.players[this.turn]
      if (target && !player.throws.includes(target)) {
        player.score = player.score + (score * player.multiplier)
        player.multiplier += 1
      }
      
      player.throws.push(target)
      this.record.push(hit)

      if (player.throws.length % 3 == 0) {
        
        if (this.turn + 1 === this.players.length && this.currentround === this.roundLimit) {
          this.gameover = true
          this.playerchange = false
          return
        } else {
          this.playerchange = true
        }
      }

      this.highlight(this.players[this.turn].throws)
    },
    calculateValue(target) {
      if (target === null) {
        return { score: 0, target: null }
      }
      const multiplierString = target[0]
      const number = target.substr(1)
      if (number == "bull") {
        return { score: 50, target: 'bull' }
      } else if (multiplierString == "o" || multiplierString == "i") {
        return { score: parseInt(number), target: 'a' + number }
      } else if (multiplierString == "d") {
        return { score: 2 * parseInt(number), target: 'a' + number }
      } else if (multiplierString == "t") {
        return { score: 3 * parseInt(number), target: 'a' + number }
      }
    },
    changeplayer() {
      if (this.turn + 1 === this.players.length) {
        this.currentround = this.currentround + 1
      }
      this.playerchange = false
      this.turn = (this.turn + 1) % this.players.length
      this.record = []
      this.highlight(this.players[this.turn].throws)
    },
  }
}
</script>

<style>

.head {
  display: grid;
  grid-template-columns: 2fr 1fr;
  place-items: center;
  border: 5px solid black;
}

.players {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  color: white;
}
.record {
  justify-self: end;
  background-color: black;
}

.record > div {
  color: white;
  font-weight: bold;
  font-size: 1.25rem;
  border: 2px solid white;
  padding: 10px 15px;
  min-width: 150px;
  min-height: 23px;
}
.players > div {
  padding: 20px 20px;
}
h3 {
  margin: 5px 0;
}
.score {
  margin-right: 20px;
}
.player {
  background-color: #333;
}
.player.active:nth-child(1) {
  background-color: red;
}
.player.active:nth-child(2) {
  background-color: blue;
}
.player.active:nth-child(3) {
  background-color: green;
}
.player.active:nth-child(3) {
  background-color: purple;
}

.playerchange, .gameover {
  background-color: rgb(126, 104, 43);
  color:white;
  font-size: 2rem;
  padding: 30px;
  cursor: pointer;
}
.gameover {
  background-color: rgb(26, 82, 74);
}

</style>
