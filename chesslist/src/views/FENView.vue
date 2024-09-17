<script setup lang="ts">
import ChessBoard from '../components/ChessBoard.vue'
import fens from '../assets/100k.fen.json'
import { ref } from 'vue'

const urlParams = new URLSearchParams(window.location.search)
const fenParam = urlParams.get('fen')

function getFEN() {
  const broken = fens[Math.floor(Math.random() * fens.length)] as string
  return `${broken} ${Math.random() > 0.5 ? 'w' : 'b'} - - 0 1`
}

function setFEN(fen: string) {
  urlParams.set('fen', fen)
  window.location.search = urlParams.toString()
}

const fen = fenParam !== null ? fenParam : getFEN()
if (fenParam === null) {
  setFEN(fen)
}

const moves = ref(0)
const errors = ref(0)

function countMoves() {
  moves.value++
  console.log(`move ${moves.value}`)
}

function countError() {
  errors.value++
  console.log(`error ${errors.value}`)
}

function finish() {
  alert(
    `You made ${moves.value} move${moves.value === 1 ? '' : 's'} with ${errors.value} error${errors.value === 1 ? '' : 's'}`
  )
  setFEN(getFEN())
}

// randomly choose a perspective
const perspective = Math.random() > 0.5 ? 'white' : 'black'
</script>
<template>
  <ChessBoard
    :fen="fen"
    :perspective="perspective"
    @move="countMoves"
    @finished="finish"
    @mistake="countError"
  />
</template>
