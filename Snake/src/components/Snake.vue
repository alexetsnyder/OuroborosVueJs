<script setup lang="ts">

import { ref, onMounted, watch } from 'vue'
import type { Ref } from 'vue'

const boardRowsDefault : number = 40
const boardColsDefault : number = 40

interface snakeDirType {
    left: number[];
    right: number[];
    up: number[];
    down: number[];
}

const snakeDirections : snakeDirType = {
    left: [-1, 0],
    right: [1, 0],
    up: [0, -1],
    down: [0, 1]
}

let intervalId : number | undefined;

let isGameOver : boolean = false;

const boardRows : Ref<number> = ref(boardRowsDefault)
const boardCols : Ref<number> = ref(boardColsDefault)
const statusText : Ref<string> = ref('')
const snakeBoard : Ref<string[]> = ref([])
const snakeQueue : Ref<number[]> = ref([])
const snakeDir : Ref<number[]> = ref(snakeDirections.left)
const appleSpawn : Ref<number> = ref(0)

const backgroundColor : string = '#013220'
const snakeColor : string = '#6CBB3C'
const appleColor : string = '#ff0000'

function SetUpBoard() : void {
    if (snakeBoard.value.length > 0) {
        snakeBoard.value.length = 0
    }

    if (!boardRows.value) {
        boardRows.value = boardRowsDefault
    }

    if (!boardCols.value) {
        boardCols.value = boardColsDefault
    }

    for (let i = 0; i < boardRows.value; i++) {
        for (let j = 0; j < boardCols.value; j++) {
            snakeBoard.value.push(backgroundColor)
        }
    }
}

function SetUpSnake() : void {
    snakeQueue.value.length = 0

    let i = Math.floor(boardRows.value / 4)
    let j = Math.floor(boardCols.value / 2 + boardCols.value / 4)

    snakeQueue.value.push(i * boardCols.value + j)
}

function GetRandomElement(array:number[]) : number {
    return array[Math.floor(Math.random() * array.length)]
} 

function SetUpAppleSpawn() : void {
    const possibleSpawns : number[] = []
    for (let i = 0; i < boardRows.value * boardCols.value; i++) {
        if (!snakeQueue.value.includes(i)) {
            possibleSpawns.push(i)
        }
    }

    appleSpawn.value = GetRandomElement(possibleSpawns)
}

function UpdateSnakeBoard() : void {
    for (let i = 0; i < snakeBoard.value.length; i++) {
        if (i === appleSpawn.value) {
            if (snakeBoard.value[i] !== appleColor) {
                snakeBoard.value[i] = appleColor
            }
        }
        else if (snakeQueue.value.includes(i)) {
            if (snakeBoard.value[i] !== snakeColor) {
                snakeBoard.value[i] = snakeColor
            }
        }
        else {
            if (snakeBoard.value[i] !== backgroundColor) {
                snakeBoard.value[i] = backgroundColor
            } 
        }
    }
}

function SetUpGame() : void {
    SetUpBoard()
    SetUpSnake()
    SetUpAppleSpawn()
    UpdateSnakeBoard()
}

let hasMoved : boolean = false;
let pastDir : number[] = snakeDir.value;

function MoveSnake() : void {
    const head = snakeQueue.value[snakeQueue.value.length - 1]
    
    let j = head % boardCols.value
    let i = (head - j) / boardCols.value

    i += snakeDir.value[1]
    j += snakeDir.value[0]

    if (i >= 0 && i < boardRows.value && j >= 0 && j < boardCols.value) {
        let index = i * boardCols.value + j

        if (snakeQueue.value.includes(index)) {
            LostGame()
        }
        else {
            snakeQueue.value.push(index)
            if (index === appleSpawn.value) {
                SetUpAppleSpawn()
            }
            else {
                snakeQueue.value.shift()
            }  
            UpdateSnakeBoard()

            hasMoved = true;

            if (snakeQueue.value.length === snakeBoard.value.length) {
                WinGame()
            }
        } 
    }
    else {
        LostGame()
    }
}

function LostGame() : void {
    Stop();
    statusText.value = 'You Lose!'
    isGameOver = true
}

function WinGame() : void {
    Stop();
    statusText.value = 'You Win!'
    isGameOver = true;
}

function Start() : void {
    if (!intervalId && !isGameOver) {
        intervalId = setInterval(MoveSnake, 160)
    } 
}

function Reset() : void {
    Stop()
    isGameOver = false;
    statusText.value = ''
    snakeDir.value = snakeDirections.left
    SetUpGame()
}

function Stop() : void {
    clearInterval(intervalId)
    intervalId = undefined;
}

function IsInputAllowed(currentDir:number[], oppDir:number[]) : boolean {
    if (currentDir[0] === oppDir[0] && currentDir[1] === oppDir[1] || 
        !hasMoved && pastDir[0] === oppDir[0] && pastDir[1] === oppDir[1]) {
        return false
    }
    return true;
}

function KeyDown(event : KeyboardEvent) : void {
    if (event.key.toLowerCase() === 'w') {
        if (IsInputAllowed(snakeDir.value, snakeDirections.down)) {
            hasMoved = false
            pastDir = snakeDir.value
            snakeDir.value = snakeDirections.up
        } 
    }
    if (event.key.toLowerCase() === 's') {
        if (IsInputAllowed(snakeDir.value, snakeDirections.up)) {
            hasMoved = false
            pastDir = snakeDir.value
            snakeDir.value = snakeDirections.down
        }  
    }
    if (event.key.toLowerCase() === 'a') {
        if (IsInputAllowed(snakeDir.value, snakeDirections.right)) {
            hasMoved = false
            pastDir = snakeDir.value
            snakeDir.value = snakeDirections.left
        }
    }
    if (event.key.toLowerCase() === 'd') {
        if (IsInputAllowed(snakeDir.value, snakeDirections.left)) {
            hasMoved = false
            pastDir = snakeDir.value
            snakeDir.value = snakeDirections.right
        }   
    }
}

SetUpGame();

watch(boardRows, () => {
    Reset()
})

watch(boardCols, () => {
    Reset()
})

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

    <div :style="{ width: boardCols * 13 + 'px' }" class="container">
        <div class="row" v-for="i in boardRows" :key="i">
            <div class="column" v-for="j in boardCols" :key="j">
                <div :class="{ cell: true, 'bottom-cell': i == boardRows, 'right-cell': j == boardCols }" :style="{ background: snakeBoard[(i - 1) * boardCols + (j - 1)] }"/>
            </div>
        </div>
    </div>

    <h1 v-if="statusText"> {{ statusText }} </h1>

    <div class="center">
        <label>Rows: </label>
        <input type="number" v-model="boardRows">
        <label> Cols: </label>
        <input type="number" v-model="boardCols">
    </div>

   <div class="center">
        <button @click="Start">Start</button>
        <button @click="Stop">Stop</button>
        <button @click="Reset">Reset</button>
   </div> 
   
</template>

<style scoped>

h1 {
    text-align: center;
    font-size: 48px;
}

.center {
    text-align: center;
    padding-top: 20px;
}

input {
    background-color: #6CBB3C;
    width: 40px;
}

button {
    background-color: #013220;
    padding: 15px 32px;
    color: white;
    text-align: center;
    text-decoration: none;
    font-size: 16px;
    margin: 4px 2px;
    cursor: pointer;
}

button:hover {
    background-color: darkgreen;
}

.container {
    margin: auto;
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
}

.column {
    flex-direction: column;
    flex: 1;
}

</style>