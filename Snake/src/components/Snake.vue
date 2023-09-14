<script setup lang="ts">

import { ref, onMounted } from 'vue'
import type { Ref } from 'vue'

const boardRows : number = 40
const boardCols : number = 40

const snakeDirections = {
    left: [-1, 0],
    right: [1, 0],
    up: [0, -1],
    down: [0, 1]
}

let intervalId : number | undefined;

let isGameOver : boolean = false;

const statusText : Ref<string> = ref('')
const snakeBoard : Ref<string[]> = ref([])
const snakeQueue : Ref<number[]> = ref([])
const snakeDir : Ref<number[]> = ref(snakeDirections.left)
const appleSpawn : Ref<number> = ref(0)

const backgroundColor : string = '#013220'
const snakeColor : string = '#6CBB3C'
const appleColor : string = '#ff0000'

function SetUpBoard() : void {
    if (snakeBoard.value.length === 0) {
        for (let i = 0; i < boardRows; i++) {
            for (let j = 0; j < boardCols; j++) {
                snakeBoard.value.push(backgroundColor)
            }
        }
    }
    else {
        for (let i = 0; i < snakeBoard.value.length; i++) {
            snakeBoard.value[i] = backgroundColor
        }
    }
}

function SetUpSnake() : void {
    snakeQueue.value.length = 0

    let i = boardRows / 4
    let j = boardCols / 2 + boardCols / 4

    snakeQueue.value.push(i * boardCols + j)
}

function GetRandomElement(array:number[]) : number {
    return array[Math.floor(Math.random() * array.length)]
} 

function SetUpAppleSpawn() : void {
    const possibleSpawns : number[] = []
    for (let i = 0; i < boardRows * boardCols; i++) {
        if (!snakeQueue.value.includes(i)) {
            possibleSpawns.push(i)
        }
    }

    appleSpawn.value = GetRandomElement(possibleSpawns)
}

function UpdateSnakeBoard() {
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

function SetUpGame() {
    SetUpBoard()
    SetUpSnake()
    SetUpAppleSpawn()
    UpdateSnakeBoard()
}

function MoveSnake() : void {
    const head = snakeQueue.value[snakeQueue.value.length - 1]
    
    let j = head % boardCols
    let i = (head - j) / boardCols

    i += snakeDir.value[1]
    j += snakeDir.value[0]

    if (i >= 0 && i < boardRows && j >= 0 && j < boardCols) {
        let index = i * boardCols + j

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

            if (snakeQueue.value.length === snakeBoard.value.length) {
                WinGame()
            }
        } 
    }
    else {
        LostGame()
    }
}

function LostGame() {
    Stop();
    statusText.value = 'You Lose!'
    isGameOver = true
}

function WinGame() {
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

function KeyDown(event : KeyboardEvent) : void {
    if (event.key === 'w') {
        snakeDir.value = snakeDirections.up
    }
    if (event.key === 's') {
        snakeDir.value = snakeDirections.down
    }
    if (event.key === 'a') {
        snakeDir.value = snakeDirections.left
    }
    if (event.key === 'd') {
        snakeDir.value = snakeDirections.right
    }
}

SetUpGame();

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

   <div class="center-button">
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

.center-button {
    text-align: center;
    padding-top: 20px;
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
    background-color: #6CBB3C;
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