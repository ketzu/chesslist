<script setup lang="ts">
import { TheChessboard, type BoardConfig, type BoardApi } from 'vue3-chessboard'
import 'vue3-chessboard/style.css'

import { Chess, type Move } from 'chess.js'
import { ref } from 'vue'

const urlParams = new URLSearchParams(window.location.search)
console.log(`Loading ${urlParams.get('fen')}`)

type MadeMove = Move & {
  done: boolean
}

const config: { api?: BoardApi } = {}

// randomly choose a perspective
// const perspective = Math.random() > 0.5 ? 'white' : 'black'
const fen = urlParams.get('fen') as string
if (fen === null) {
  throw new Error('No FEN provided')
}
const game = new Chess(fen)
const perspective = game.turn() === 'w' ? 'white' : 'black'

const moves = ref<{ checks: MadeMove[]; captures: MadeMove[]; all: MadeMove[] }>({
  checks: [],
  captures: [],
  all: []
})

game
  .moves({
    verbose: true
  })
  .forEach((move) => {
    if (move.color !== perspective[0]) return
    const mademove = { ...move, done: false }
    moves.value.all.push(mademove)
    if (move.captured !== undefined) {
      moves.value.captures.push(mademove)
    }
    if (game.move(move)) {
      if (game.isCheck()) {
        moves.value.checks.push(mademove)
      }
      game.undo()
    }
  })

console.log(moves.value)

const boardConfig: BoardConfig = {
  events: {
    move: (from, to, _capture) => {
      const move = moves.value.all.find((m) => m.from === from && m.to === to)
      if (move !== undefined) {
        move.done = true
      }

      setTimeout(() => {
        config.api?.undoLastMove()
      }, 1000)
    }
  },
  fen,
  orientation: perspective,
  coordinates: true
}

function setAPI(api: BoardApi) {
  config.api = api
}
</script>
<template>
  <div class="container">
    <div>
      <TheChessboard :board-config="boardConfig" @board-created="setAPI" />
    </div>
    <div class="info">
      <h1>Chesslist</h1>
      <h2>Playing {{ perspective }}</h2>
      <h2>Checks ({{ moves.checks.filter((m) => m.done).length }}/{{ moves.checks.length }})</h2>
      <ul>
        <li v-for="move in moves.checks.filter((m) => m.done)" :key="move.after">
          {{ move.san }}
        </li>
      </ul>
      <h2>
        Captures ({{ moves.captures.filter((m) => m.done).length }}/{{ moves.captures.length }})
      </h2>
      <ul>
        <li v-for="move in moves.captures.filter((m) => m.done)" :key="move.after">
          {{ move.san }} {{ move.captured }}
        </li>
      </ul>
    </div>
  </div>
</template>
<style scoped>
.container {
  display: flex;
  align-items: center;
  height: 100%;
}
.container > div {
  flex: 1;
}
.info {
  flex: 1;
  padding-left: 2rem;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  justify-content: flex-start;
  gap: 1rem;
}
ul,
li {
  list-style-type: none;
  padding: 0;
  margin: 0;
}
</style>
