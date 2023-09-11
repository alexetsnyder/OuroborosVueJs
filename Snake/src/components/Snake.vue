<script setup lang="ts">

import { ref } from 'vue'
import type { Ref } from 'vue'

const boardRows : number = 40
const boardCols : number = 40

const board : Ref<string[]> = ref([])
const queue : Ref<number[]> = ref([])

const grey = '#808080'
const red = '#ff'

function CreateBoard() {
    for (let i = 0; i < boardRows; i++) {
        for (let j = 0; j < boardCols; j++) {
            board.value.push('#ff0000')
        }
    }
}
CreateBoard()

</script>

<template>
    <h1>Snake!</h1>

    <div>
        <div class="container">
            <div class="row" v-for="i in boardRows" :key="i">
                <div class="column" v-for="j in boardCols" :key="j">
                    <div class="cell bottom-cell right-cell" v-if="i == boardRows && j == boardCols">

                    </div>
                    <div class="cell bottom-cell" v-else-if="i == boardRows">
                    
                    </div>
                    <div class="cell right-cell" v-else-if="j == boardCols">

                    </div>
                    <div class="cell" v-else>

                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>

h1 {
    text-align: center;
}

.container {
    transform: translateX(136%);
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
}

.column {
    flex-direction: column;
    flex: 1;
}

</style>