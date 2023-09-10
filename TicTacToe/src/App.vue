<script setup lang="ts">

import { ref, computed } from 'vue'

const player = ref('X')
const board = ref([
	['', '', ''],
	['', '', ''],
	['', '', '']
])

function CheckForWinner(flatBoard:string[]) {
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

        if (GetPossibleMoves(board.value).length > 0) {
            ComputerMove();
        }	
	}
}

function ComputerMove() {
	if (player.value === 'O')
	{
		if (winner.value) {
			return
		}

		let move = GetComputerMove()

        if (move) {
            let [i, j] = move

            board.value[i][j] = player.value;

            player.value = 'X'
        }
	}
}

function GetComputerMove() {
    return Evaluate(board.value, player.value)
}

function GetPossibleMoves(array:string[][]) {
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

function GetArrayCopies(array:string[][], count:number)
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

interface Branch  {
    moves: number[][];
    winner: string | null;
}

interface Move {
    move: number[];
    wins: number;
    loses: number;
    ties: number;
}

function CreateBranch(moves:number[][], winner:string) : Branch {
    return { moves: moves, winner: winner }
}

function Evaluate(array:string[][], fPlayer:string) : number[] {
    const possibleMoves = GetPossibleMoves(array)

    const arrayCopies = GetArrayCopies(array, possibleMoves.length)

    const branches : Branch[] = []
    for (let i = 0; i < possibleMoves.length; i++) {
        if (EvaluateMove(arrayCopies[i], possibleMoves[i], fPlayer)) {
            return possibleMoves[i]
        }
        else {
            let possibleBranch = CreateBranch([possibleMoves[i]], '');
            let bs = EvaluateRecursive(possibleBranch, arrayCopies[i], 'X')
            branches.push(...bs)
        }
    }

    const moves : Move[] = EvaluateBranches(branches)

    CullMoves(moves)

    let maxChance = 0
    let maxMove : Move = moves[0]
    for (let i = 0; i < moves.length; i++) {
        let move = moves[i]
        let chance = move.wins / move.loses
        if (chance > maxChance) {
            maxChance = chance
            maxMove = move
        }
    }
    
    return maxMove.move
}

function CullMoves(moves:Move[]) {
    const removeArray = []
    for (let i = 0; i < moves.length; i++) {
        let move = moves[i]
        if (move.wins === 0 && move.ties === 0 && move.loses > 0) {
            removeArray.push(move)
        }
    }

    if (removeArray.length < moves.length) {
        for (let i = 0; i < removeArray.length; i++) {
            let index = moves.indexOf(removeArray[i])
            moves.splice(index, 1)
        }
    } 
}

function EvaluateBranches(branches:Branch[]) : Move[] {
    const moves : Move[] = []
    for (let i = 0; i < branches.length; i++) {
        let branch = branches[i]
        let isFound = false
        for (let j = 0; j < moves.length; j++) {
            let move = moves[j]
            if (move.move === branch.moves[0]) {
                EvaluateBranch(branch, move)
                isFound = true;
                break;
            }
        }

        if (!isFound) {
            let move : Move = {
                move: branch.moves[0],
                wins: 0,
                loses: 0,
                ties: 0,
            }
            EvaluateBranch(branch, move)
            moves.push(move)
        }
    }
    
    return moves
}

function EvaluateBranch(branch:Branch, move:Move) {
    if (branch.winner === 'O') {
        move.wins++
    }
    else if (branch.winner === 'X') {
        move.loses++
    }
    else {
        move.ties++
    }
}

function EvaluateRecursive(possibleBranch: Branch, array:string[][], fPlayer:string) : Branch[] {
    const possibleMoves = GetPossibleMoves(array)

    if (possibleMoves.length == 0) {
        return [possibleBranch]
    }

    const arrayCopies = GetArrayCopies(array, possibleMoves.length)

    const branches : Branch[] = []
    for (let i = 0; i < possibleMoves.length; i++) {
        if (EvaluateMove(arrayCopies[i], possibleMoves[i], fPlayer)) {
            return [{ moves: possibleBranch.moves.concat([possibleMoves[i]]), winner: fPlayer }]
        }
        else {
            let newPossibleBranch : Branch = {
                moves: possibleBranch.moves.concat([possibleMoves[i]]),
                winner: possibleBranch.winner
            }
            branches.push(...EvaluateRecursive(newPossibleBranch, arrayCopies[i], (fPlayer === 'X') ? 'O' : 'X'))
        }
    }

    return branches
}

function EvaluateMove(array:string[][], move:number[], fPlayer:string) {
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
