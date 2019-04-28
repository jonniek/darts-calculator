<template>
  <div id="app">
    <div v-if="!gamestarted">
      <label>
        Player count
        <input type="number" v-model="players">
      </label>
      <label>
        Game mode, 0: shootout, 1: zero1
        <input type="number" v-model="game">
      </label>
      <Button @click="gamestarted = true">Start</Button>
    </div>
    <div v-else>
      <shootout v-if="game == 0" v-bind:playercount="players" />
      <zero1 v-if="game == 1"  v-bind:playercount="players" />
    </div>
  </div>
</template>

<script>
import shootout from './shootout.vue'
import zero1 from './zero1.vue'

export default {
  name: 'app',
  components: {
    shootout,
    zero1,
  },
  data: function() {
    return {
      game: 0,
      players: 1,
      gamestarted: false,
    }
  },
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: left;
  color: #2c3e50;
  margin-top: 60px;
}
label { display: block;  margin: 10px 0 }
input { display: block; }
</style>
