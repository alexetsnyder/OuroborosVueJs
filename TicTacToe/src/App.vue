<script setup lang="ts">

import { ref, computed } from 'vue'

const player = ref('X')
const board = ref([
  ['', '', ''],
  ['', '', ''],
  ['', '', '']
])

function CheckForWinner(flatBoard:Array<string>) {
  const checks = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 4, 8],
    [2, 4, 6],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8]
  ]

  for (let i = 0; i < checks.length; i++) {
    let [j, k, l] = checks[i]
    if (flatBoard[j] && flatBoard[j] === flatBoard[k] && flatBoard[k] === flatBoard[l])
    {
      return flatBoard[j]
    }
  }

  return null
}

let winner = computed(() => CheckForWinner(board.value.flat()))

function MakeMove(x:number, y:number) {
  if (winner.value) {
    return
  }

  if (board.value[x][y] !== '') {
    return;
  }

  board.value[x][y] = player.value;

  player.value = (player.value === 'X') ? 'O' : 'X'
}

function Reset() {
  board.value = [
    ['', '', ''],
    ['', '', ''],
    ['', '', '']
  ]
  player.value = 'X'
}

</script>

<template>
  <main>
    
    <h1>Tic Tac Toe!</h1>

    <div class="flex-container" v-for="(row, i) in board" :key="i">
      <div v-for="(val, j) in row" :key="j">
        <div @click="MakeMove(i, j)" class="cell flex-child">
          {{ val }}
        </div>  
      </div>
    </div>

    <h2 v-if="winner">Player {{ winner }} has Won!</h2>

    <button style="margin-top: 10px;" @click="Reset()">Reset</button>

  </main>
</template>

<style scoped>

main {
  text-align: center;
}

.cell {
  width: 100px;
  height: 100px;
  border-style: solid;
  border-color: black;
  font-size: 100px;
}

.cell:hover {
  background-color: lightgray;
}

.flex-container {
  display: flex;
  justify-content: center;
}

.flex-child {
  flex: 1;
}

</style>
