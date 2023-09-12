<script setup lang="ts">

import { ref } from 'vue'
import type { Ref } from 'vue'

const boardRows : number = 40
const boardCols : number = 40

const board : Ref<string[]> = ref([])
const queue : Ref<number[]> = ref([])

function CreateBoard() {
    for (let i = 0; i < boardRows; i++) {
        for (let j = 0; j < boardCols; j++) {
            board.value.push('#808080')
        }
    }
}
CreateBoard()

</script>

<template>
    <h1>Snake!</h1>

    <div class="container">
        <div class="row" v-for="i in boardRows" :key="i">
            <div class="column" v-for="j in boardCols" :key="j">
                <div :class="{ cell: true, 'bottom-cell': i == boardRows, 'right-cell': j == boardCols }" :style="{ background: board[(i - 1) * boardCols + (j - 1)] }"/>
            </div>
        </div>
    </div>
</template>

<style scoped>

h1 {
    text-align: center;
}

.container {
    margin: auto;
    width: 520px;
}

.cell {
    border-top: solid;
    border-left: solid;
    width: 10px;
    height: 10px;
}

.bottom-cell {
    border-bottom: solid;
}

.right-cell {
    border-right: solid;
}

.row {
    display: flex;
    flex-direction: row;
    flex-wrap: nowrap;
    width: 520px;
}

.column {
    flex-direction: column;
    flex: 1;
}

</style>