<template>
  <div id="app">
    <div v-if="game == 0" class="menu">
      <h2>Select game mode</h2>
      <div class="gamemode shootout" @click="game = 1">Shoot out</div>
      <div class="gamemode zero1" @click="game = 2; zero1score = 301">301</div>
      <div class="gamemode zero12" @click="game = 2; zero1score = 501">501</div>
      <div class="gamemode zero13" @click="game = 2; zero1score = 701">701</div>

      <div class="highscore" v-if="shootoutScores.length > 0">
        <h3>Shootout highscores</h3>
        <hr />
        <div v-for="(score, index) in shootoutScores" :key="index" class="line">
          <div class="name">{{score.name}}</div>
          <div class="score">{{score.score}}</div>
        </div>
      </div>
    </div>
    <div v-else-if="players.length == 0">
      <h2>Select player count</h2>
      <div class="playercount">
        <div @click="setPlayers(1)">1</div>
        <div @click="setPlayers(2)">2</div>
        <div @click="setPlayers(3)">3</div>
        <div @click="setPlayers(4)">4</div>
        <div @click="setPlayers(5)">5</div>
      </div>
    </div>
    <div v-else-if="!gamestarted">
      <h2>Edit player names</h2>
      <div class="playernames">
        <input v-for="index in players.length" v-model="players[index - 1]" :key="index">
      </div>
      <div class="startgame" @click="gamestarted = true">Start game</div>
    </div>
    <div v-else>
      <shootout v-if="game == 1" v-bind:playernames="players" v-on:saveScore="handleAddScore" v-on:gameover="reset"/>
      <zero1 v-if="game == 2"  v-bind:playernames="players" :initialScore="zero1score" v-on:gameover="reset"/>
    </div>
  </div>
</template>

<script>
import shootout from './shootout.vue'
import zero1 from './zero1.vue'
import './assets/global.css'

export default {
  name: 'app',
  components: {
    shootout,
    zero1,
  },
  data: function() {
    return {
      game: 0,
      players: [],
      gamestarted: false,
      highscores: [],
      zero1score: 301,
    }
  },
  created() {
    if (localStorage) {
      this.highscores = JSON.parse(localStorage.getItem('highscore') || '[]')
    }
  },
  computed: {
    shootoutScores: function() {
      return this.highscores.filter(s => s.mode === 'shootout').sort((a, b) => a.score < b.score).slice(0, 5)
    },
  },
  methods: {
    handleAddScore(score) {
      this.highscores.push(score)
      if (localStorage) {
        const scores = JSON.parse(localStorage.getItem('highscore') || '[]')
        localStorage.setItem('highscore', JSON.stringify([...scores, score]))
      }
    },
    reset() {
      this.game = 0
      this.players = 0
      this.gamestarted = false
    },
    setPlayers(count) {
      const players = []
      for (let i = 0; i < count; i++) {
        players.push("player " + (i+1))
      }
      this.players = players
    }
  }
}
</script>

<style scoped>
.gamemode, .playercount > div, .startgame {
  color: white;
  font-weight: bold;
  padding: 20px 30px;
  cursor: pointer;
}
.shootout {
  background-color: rgb(145, 26, 26)
}
.zero1 {
  background-color: rgb(54, 123, 128)
}
.zero12 {
  background-color: rgb(125, 24, 134)
}
.zero13 {
  background-color: rgb(79, 155, 35)
}
.playercount > div:nth-child(1) {
  background-color: red;
}
.playercount > div:nth-child(2) {
  background-color: blue;
}
.playercount > div:nth-child(3) {
  background-color: green;
}
.playercount > div:nth-child(4) {
  background-color: purple;
}
.playercount > div:nth-child(5) {
  background-color: rgb(28, 114, 110);
}
.highscore {
  padding: 30px;
}
.highscore > h3 {
  margin-bottom: 30px;
}
.line {
  display: flex;
  font-weight: bold;
  padding: 10px;
}
.line:nth-child(even) {
  background-color: #f0f0f0;
}
.name {
  color: #333;
}
.score {
  color: rgb(45, 121, 45);
  margin-left: auto;
}

.playernames {
  display: flex;
  flex-direction: column;
  padding: 30px;
  gap: 20px;
}
.playernames > input {
  max-width: 300px;
}
.startgame {
  background-color: #222
}


</style>
