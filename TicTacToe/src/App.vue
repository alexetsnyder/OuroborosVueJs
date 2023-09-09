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

	if (player.value === 'X')
	{
		board.value[x][y] = player.value;

		player.value = 'O'

		ComputerMove();
	}
}

function ComputerMove() {
	if (player.value === 'O')
	{
		if (winner.value) {
			return
		}

		let [i, j] = GetComputerMove()

        if (i === -1 && j === -1) {
            return
        }

		board.value[i][j] = player.value;

	    player.value = 'X'
	}
}

function GetPossibleMoves(array:Array<Array<string>>) {

    let possibleMoves = []

    for (let i = 0; i < array.length; i++)
    {
        for (let j = 0; j < array[i].length; j++)
        {
            if (array[i][j] === '') {
                possibleMoves.push([i, j])
            }
        }
    }

    return possibleMoves
}

function GetArrayCopies(array:Array<Array<string>>, count:number)
{
    const arrayCopies = []
    for (let i = 0; i < count; i++)
    {
        let arrayCopy = [
		    ['', '', ''],
		    ['', '', ''],
		    ['', '', '']
	    ]

        for (let i = 0; i < array.length; i++) {
            for (let j = 0; j < array[i].length; j++) {
                arrayCopy[i][j] = array[i][j];
            }
        }

        arrayCopies.push(arrayCopy)
    }
    return arrayCopies
}

function GetComputerMove() {
    let ret = EvaluateRecursive(board.value, player.value)
    if (ret) {
        return ret
    }
    return [-1, -1]
}

function EvaluateRecursive(array:Array<Array<string>>, fPlayer:string) {
    const possibleMoves = GetPossibleMoves(array)

    const arrayCopies = GetArrayCopies(array, possibleMoves.length)

    const branches = []
    for (let i = 0; i < possibleMoves.length; i++) {
        if (EvaluateMove(arrayCopies[i], possibleMoves[i], fPlayer)) {
            return (fPlayer === 'O') ? possibleMoves[i] : null
        }
        else {
            let branch : {
                move: Array<number>;
                game: Array<number> | null;
            }

            branch = {
                move: possibleMoves[i],
                game: EvaluateRecursive(arrayCopies[i], (fPlayer === 'X') ? 'O' : 'X')
            }
            branches.push(branch)
        }
    }

    for (let i = 0; i < branches.length; i++) {
        if (branches[i].game) {
            return branches[i].move
        }       
    }

    return (branches.length > 0) ? branches[0].move : null
}

function EvaluateMove(array:Array<Array<string>>, move:Array<number>, fPlayer:string) {
    let [i, j] = move

    array[i][j] = fPlayer

    if (CheckForWinner(array.flat())) {
        return fPlayer
    }

    return null
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
