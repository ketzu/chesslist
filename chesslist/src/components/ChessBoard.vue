<script setup lang="ts">
import { TheChessboard, type BoardConfig, type BoardApi } from 'vue3-chessboard'
import 'vue3-chessboard/style.css'

import { Chess, type Move } from 'chess.js'
import { ref } from 'vue'

const emit = defineEmits(['move', 'finished', 'mistake'])
const props = defineProps(['fen', 'perspective'])

type MadeMove = Move & {
  done: boolean
}
const config: { api?: BoardApi } = {}

const fen = props.fen
const perspective = props.perspective

const game = new Chess(fen)

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
    if (move.color !== game.turn()) return
    const mademove = { ...move, done: false }
    const capture = move.captured !== undefined

    const check = game.move(move) && game.isCheck()
    game.undo()

    if (capture) moves.value.captures.push(mademove)
    if (check) moves.value.checks.push(mademove)
    if (check || capture) {
      moves.value.all.push(mademove)
    }
  })

const boardConfig: BoardConfig = {
  events: {
    move: (from, to) => {
      const move = moves.value.all.find((m) => m.from === from && m.to === to)
      emit('move', from, to)
      if (move !== undefined) {
        move.done = true
      } else {
        emit('mistake')
      }

      console.log(moves.value.all.every((m) => m.done))
      console.log(moves.value.all)

      if (moves.value.all.every((m) => m.done)) {
        emit('finished')
      }

      setTimeout(() => {
        config.api?.undoLastMove()
      }, 250)
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
      <h2>Playing {{ game.turn() ? 'White' : 'Black' }}</h2>
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
      <button @click="emit('finished')">Give Up</button>
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
