<template>
  <div id="shootout">
    <input v-model="message" placeholder="">
    <button @click="handleDartHit">submit</button>
    <h2>{{ players[turn].name }} turn to throw</h2>
    <h2 v-for="player in players" v-bind:key="player.name">
      {{ player.name }} - score: {{ player.score }} - mult: {{ player.multiplier }}
    </h2>
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
      message: "",
      turn: 0,
      roundLimit: 8,
      players: [],
    }
  },
  created() {
    this.init()
  },
  methods: {
    init() {
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
      const { score, target } = this.calculateValue(hit || this.message)
      const player = this.players[this.turn]
      if (!player.throws.includes(target)) {
        player.score = player.score + (score * player.multiplier)
        player.multiplier += 1
      }
      
      player.throws.push(target)

      if (player.throws.length % 3 == 0) {
        this.turn = (this.turn + 1) % this.players.length
      }

      this.highlight(this.players[this.turn].throws)
    },
    calculateValue(target) {
      if (target === 'miss') {
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
