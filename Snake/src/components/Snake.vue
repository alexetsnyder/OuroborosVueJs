<script setup lang="ts">

import { ref, computed, onMounted } from 'vue'
import type { Ref, ComputedRef } from 'vue'

const boardRows : number = 40
const boardCols : number = 40

const snakeDirections = {
    left: [-1, 0],
    right: [1, 0],
    up: [0, 1],
    down: [0, -1]
}

const board : Ref<string[]> = ref([])
const snakeQueue : Ref<number[]> = ref([])
const snakeDir : Ref<number[]> = ref(snakeDirections.left)
const appleSpawn : Ref<number> = ref(0)

const backgroundColor : string = '#808080'
const snakeColor : string = '#6CBB3C'
const appleColor : string = '#ff0000'

function CreateBoard() : void {
    for (let i = 0; i < boardRows; i++) {
        for (let j = 0; j < boardCols; j++) {
            board.value.push(backgroundColor)
        }
    }
}
CreateBoard()

function CreateSnake() : void {
    let i = boardRows / 4
    let j = boardCols / 2 + boardCols / 4

    snakeQueue.value.push(i * boardCols + j)
}
CreateSnake()

function MoveSnake() : void {
    const head = snakeQueue.value[snakeQueue.value.length - 1]
    
    let j = head % boardCols
    let i = (head - j) / boardCols

    i += snakeDir.value[0]
    j += snakeDir.value[1]

    if (i >= 0 && i < boardRows && j >= 0 && j < boardCols) {
        snakeQueue.value.shift()
        snakeQueue.value.push(i * boardCols + j)
    }
}

function GetRandomElement(array:number[]) : number {
    return array[Math.floor(Math.random() * array.length)]
} 

function GetAppleSpawn() : void {
    const possibleSpawns : number[] = []
    for (let i = 0; i < board.value.length; i++) {
        if (!snakeQueue.value.includes(i)) {
            possibleSpawns.push(i)
        }
    }

    appleSpawn.value = GetRandomElement(possibleSpawns)
}
GetAppleSpawn()

const snakeBoard : ComputedRef<string[]> = computed(() => {
    const newBoard : string[] = []
    for (let i = 0; i < board.value.length; i++) {
        if (i === appleSpawn.value) {
            newBoard.push(appleColor)
        }
        else if (snakeQueue.value.includes(i)) {
            newBoard.push(snakeColor)
        }
        else {
            newBoard.push(backgroundColor)
        }
    }
    return newBoard
})

function KeyDown(event : Event) : void {
    console.log(event)
}

onMounted(() => {
    window.addEventListener('keydown', (e) => {
        if (e.key === 'Tab') {
            e.preventDefault()
        }
        KeyDown(e)
    })
})

</script>

<template>
    <h1>Snake!</h1>

    <div class="container">
        <div class="row" v-for="i in boardRows" :key="i">
            <div class="column" v-for="j in boardCols" :key="j">
                <div :class="{ cell: true, 'bottom-cell': i == boardRows, 'right-cell': j == boardCols }" :style="{ background: snakeBoard[(i - 1) * boardCols + (j - 1)] }"/>
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