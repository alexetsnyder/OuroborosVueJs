<script setup lang="ts">

import { ref, computed } from 'vue'

const player = ref('X')
const board = ref([
  ['', '', ''],
  ['', '', ''],
  ['', '', '']
])

</script>

<template>
  <main>
    
    <h1>Tic Tac Toe!</h1>

    <div class="flex-container" v-for="(row, i) in board" :key="i">
      <div v-for="(val, j) in row" :key="j">
        <div class="cell flex-child">
          {{ val }}
        </div>
        
      </div>
    </div>

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
