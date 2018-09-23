<template>
  <div id="app">
    <input v-model="message" placeholder="">
    <button @click="handleDartHit">submit</button>
    <h2>{{ players[turn].name }} turn to throw</h2>
    <h2 v-for="player in players" v-bind:key="player.name">{{ player.name }}: {{ player.score }}</h2>
    <Dartboard />
  </div>
</template>

<script>
import Dartboard from './components/Dartboard.vue'

export default {
  name: 'app',
  components: {
    Dartboard
  },
  data: function() {
    return {
      message: "",
      turn: 0,
      players: null,
    }
  },
  created() {
    this.init()
  },
  methods: {
    init(playercount = 2, score = 301) {
      this.players = [...Array(playercount).keys()].map(index => {
        return { name: "player " + (index+1), score, throws: [] }
      })
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
    handleDartHit() {
      const value = this.calculateValue(this.message)
      const player = this.players[this.turn]
      player.throws.push(value)

      if (player.throws.length % 3 == 0) {
        this.turn = (this.turn + 1) % this.players.length
      }
      
      if (player.score - value < 0) {
        console.log("Busted")
        return
      }

      player.score = player.score - value
      console.log("Scored: ", value)

      if (player.score == 0) {
        console.log(player.name, "won!")
        this.init()
      }
      this.highlight(this.players[this.turn].score)

    },
    calculateValue(target) {
      const multiplierString = target[0]
      const number = target.substr(1)
      if (number == "bull") {
        return 50
      } else if (multiplierString == "o" || multiplierString == "i") {
        return parseInt(number)
      } else if (multiplierString == "d") {
        return 2 * parseInt(number)
      } else if (multiplierString == "t") {
        return 3 * parseInt(number)
      }
      return 0
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
