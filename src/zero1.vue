<template>
  <div id="shootout">
    <div class="head">
      <h2>{{ this.currentround }}/{{ this.roundLimit }}</h2>
      <div class="record">
        <div @click="undo">{{ record[0] | totext }}</div>
        <div>{{ record[1] | totext }}</div>
        <div>{{ record[2] | totext }}</div>
      </div>
    </div>
    <div class="players">
      <div v-for="(player, index) in players" v-bind:key="player.name" :class="{ active: turn === index, player }">
        <h3>{{ player.name }}</h3>
        <span class="score">{{ player.score }}</span>
      </div>
    </div>
    <div v-if="playerchange" class="playerchange" @click="changeplayer">Player change</div>
    <div v-if="gameover" class="gameover">
      <div class="mainmenu" @click="goToMenu">Main menu</div>
      <div class="playagain" @click="init">Play again</div>
    </div>
    <Dartboard v-on:hit="handleDartHit" :disabled="playerchange || gameover"/>
  </div>
</template>

<script>
import Dartboard from './components/Dartboard.vue'

export default {
  name: 'zero1',
  components: {
    Dartboard
  },
  props: ['playernames', 'initialScore'],
  data: function() {
    return {
      turn: 0,
      roundLimit: 20,
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
      this.playerchange = false
      this.record = []
      this.currentround = 1
      this.turn = 0
      this.players = this.playernames.map(name => ({
        name,
        score: this.initialScore || 301,
        throws: [],
      }))
    },
    highlight(target) {
      document.querySelectorAll('.bull,.a20,.a19,.a18,.a17,.a16,.a15,.a14,.a13,.a12,.a11,.a10,.a9,.a8,.a7,.a6,.a5,.a4,.a3,.a2,.a1')
        .forEach(item => item.style.fill = "inherit")
      const range = [...Array(20).keys()].map(i => i+1)
      let validTargets;
      if (target == 50) {
        validTargets = document.querySelectorAll(".bull")
      } else {
        const validSingles = range.filter(i => i == target).map(s => ".s"+s)
        const validDoubles = range.filter(i => i*2 == target).map(s => "#d"+s)
        const validTriples = range.filter(i => i*3 == target).map(s => "#t"+s)
        const valids = validSingles.concat(validDoubles).concat(validTriples)
        const query = valids.reduce((selector, item) => {
          if (selector !== "") selector += ","
          selector += item
          return selector
        }, "")
        validTargets = query == "" ? [] : document.querySelectorAll(query)
      }
      validTargets.forEach(t => t.style.fill = "green")
    },
    handleDartHit(hit) {
      const { score, target } = this.calculateValue(hit)
      const player = this.players[this.turn]
      
      player.throws.push(target)

      if (player.score - score === 0) {
        player.score = player.score - score
        this.record.push(hit)
        this.gameover = true
      } else if (player.score - score > 0) {
        player.score = player.score - score
        this.record.push(hit)
      } else {
        this.record.push('bust')
      }

      if (player.throws.length % 3 == 0) {
        
        if (this.turn + 1 === this.players.length && this.currentround === this.roundLimit) {
          this.gameover = true
          this.playerchange = false
          return
        } else {
          this.playerchange = true
        }
      }

      this.highlight(this.players[this.turn].score)
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
      this.highlight(this.players[this.turn].score)
    },
    undo() {
      const player = this.players[this.turn]
      const recordCount = this.record.length
      const scoreToUndo = this.record.reduce((sum, next) => sum + this.calculateValue(next).score, 0)
      player.throws = player.throws.slice(0, recordCount * -1)
      player.score += scoreToUndo
      this.record = []
      this.playerchange = false
      this.highlight(this.players[this.turn].score)
    },
    goToMenu() {
      this.$emit('gameover')
    }
  }
}
</script>

<style>
#shootout { display: relative }
.head {
  display: grid;
  grid-template-columns: 2fr 1fr;
  place-items: center;
  border: 5px solid black;
}

.players {
  display: flex;
  color: white;
}
.record {
  justify-self: end;
  background-color: black;
}

.record > div {
  color: white;
  font-weight: bold;
  font-size: 1rem;
  border: 2px solid white;
  padding: 10px;
  min-width: 150px;
  min-height: 45px;
}
.players > div {
  padding: 10px;
  border-left: 4px solid black;
  border-bottom: 4px solid black;
}.players > div:last-child {
  border-right: 4px solid black;
}
h3 {
  margin: 0 0 5px 0;
}
.score {
  margin-right: 10px;
}
.player {
  background-color: #333;
  flex-grow: 1;
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
.player.active:nth-child(4) {
  background-color: purple;
}
.player.active:nth-child(5) {
  background-color: rgb(28, 114, 110);
}

.playerchange, .gameover {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 100%;
  font-weight: bold;
  transform: translate(-50%, -50%);
  background-color: rgb(189, 201, 87);
  color:white;
  font-size: 2rem;
  padding: 30px;
  cursor: pointer;
}
.gameover { padding: 0}
.gameover > div {padding: 30px}
.playagain {
  background-color: rgb(45, 102, 19);
}

</style>
