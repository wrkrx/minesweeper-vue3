<script setup>
import CellComp from './Cell.vue'
import { ref } from 'vue'
import { cellState } from '@/stores/enums.js'

const rowsCount = 16
const colsCount = 16

const maxRow = rowsCount - 1
const maxCol = colsCount - 1

const cellsCount = rowsCount * colsCount
const maxCell = cellsCount - 1

const minesCount = 20

const nonMinedCellsCount = cellsCount - minesCount

/* return a random integer between the specified values (inclusive) */
const randInt = (min, max) => {
	min = Math.ceil(min)
	max = Math.floor(max)
	return Math.floor(Math.random() * (max - min + 1) + min)
}

/* Return a copy of the array in shuffled order */
const shuffle = (array) => {
	// copy the array
	const shuffledArray = [...array]

	for (var index = shuffledArray.length - 1; index >= 1; index--) {
		const randomIndex = randInt(0, index)
		// swap values at index index and randomIndex
		const tempVal = shuffledArray[index]
		shuffledArray[index] = shuffledArray[randomIndex]
		shuffledArray[randomIndex] = tempVal
	}
	return shuffledArray
}

const getIndexFromCoords = (column, row) => {
	return rowsCount * row + column
}

const getAdjacentCellsIndices = (index) => {
	let result = []

	const column = index % rowsCount
	const row = Math.floor(index / rowsCount)

	if (row > 0) {
		// check upper cell
		result.push(getIndexFromCoords(column, row - 1))
		// check upper left cell
		if (column > 0) result.push(getIndexFromCoords(column - 1, row - 1))
		// check upper right cell
		if (column < maxCol) result.push(getIndexFromCoords(column + 1, row - 1))
	}

	// check cell directly to the left
	if (column > 0) result.push(getIndexFromCoords(column - 1, row))
	// check cell directly to the right
	if (column < maxCol) result.push(getIndexFromCoords(column + 1, row))

	if (row < maxRow) {
		// check lower cell
		result.push(getIndexFromCoords(column, row + 1))
		// check lower left cell
		if (column > 0) result.push(getIndexFromCoords(column - 1, row + 1))
		// check lower right cell
		if (column < maxCol) result.push(getIndexFromCoords(column + 1, row + 1))
	}

	return result
}

const cellsInit = Array(cellsCount)

// build list of default cells
for (let i = maxCell; i >= 0; i--) {
	cellsInit[i] = {
		mined: false,
		state: cellState.fresh,
		adjacentMinesCount: 0
	}
}

// place mines by shuffling cells and selecting x first cells
const shuffledCells = shuffle(cellsInit)
const selectedCells = shuffledCells.slice(0, minesCount)
for (let i = selectedCells.length - 1; i >= 0; i--) {
	selectedCells[i].mined = true
}

// compute adjacent mines number for each cells
for (let i = maxCell; i >= 0; i--) {
	const adjacentIndices = getAdjacentCellsIndices(i)
	for (let j = adjacentIndices.length - 1; j >= 0; j--) {
		const adjacentIndex = adjacentIndices[j]
		if (adjacentIndex >= 0 && adjacentIndex < cellsCount) {
			if (cellsInit[adjacentIndex].mined) {
				cellsInit[i].adjacentMinesCount++
			}
		}
	}
}

const cells = ref(cellsInit)
let gameStarted = ref(false)
let gameOver = ref(false)
let gameOverMessage = ref(undefined)
let gameOverGifUrl = ref(undefined)
let markedCellsCount = ref(minesCount)
let timer = ref(0)
let timerId = undefined

const revealCellByIndex = (cellIndex, shouldCheckWinCondition = true) => {
	if (gameOver.value) return
	if (!gameStarted.value) {
		gameStarted.value = true
		timerId = setInterval(function () {
			if (timer.value < 999) timer.value = timer.value + 1
		}, 1000)
	}

	const cell = cells.value[cellIndex]

	if (cell.state == cellState.revealed) return
	if (cell.state == cellState.marked) return

	cell.state = cellState.revealed

	if (cell.mined) {
		loose()
		return
	}

	if (cell.adjacentMinesCount == 0) {
		const adjacentIndices = getAdjacentCellsIndices(cellIndex)
		for (let i = adjacentIndices.length - 1; i >= 0; i--) {
			const adjacentIndex = adjacentIndices[i]
			revealCellByIndex(adjacentIndex, false)
		}
	}

	if (shouldCheckWinCondition && checkWinCondition()) {
		win()
		return
	}
}
const markCellByIndex = (cellIndex) => {
	if (gameOver.value) return

	const cell = cells.value[cellIndex]

	if (cell.state == cellState.revealed) return

	if (cell.state == cellState.marked) {
		cell.state = cellState.fresh
		markedCellsCount.value++
	} else {
		if (markedCellsCount.value) {
			cell.state = cellState.marked
			markedCellsCount.value--
		}
	}
}
const checkWinCondition = () => {
	let nonMinedRevealedCellsCount = 0
	for (var i = cells.value.length - 1; i >= 0; i--) {
		const cell = cells.value[i]
		if (!cell.mined && cell.state == cellState.revealed) {
			nonMinedRevealedCellsCount++
		}
	}

	return nonMinedCellsCount == nonMinedRevealedCellsCount
}
const win = () => {
	gameOver.value = true
	clearInterval(timerId)
	markedCellsCount.value = 0
	setTimeout(function () {
		gameOverMessage.value = 'You win!'
		displayRandomGifFromKeyword('victory')
	}, 500)
}
const loose = () => {
	gameOver.value = true
	clearInterval(timerId)
	setTimeout(function () {
		gameOverMessage.value = 'BOOOOOOOOM!!!'
		displayRandomGifFromKeyword('explosion')
	}, 500)
}

const displayRandomGifFromKeyword = (keyword) => {
	// set the apikey and limit
	const apikey = 'LIVDSRZULELA'
	const limit = 20
	// using default locale of en_US
	var search_url = `https://g.tenor.com/v1/search?q=${keyword}&key=${apikey}&limit=${limit}`

	// request the gif
	httpGetAsync(search_url, (responsetext) => {
		const randomGifIndex = Math.floor(Math.random() * limit)
		var response_objects = JSON.parse(responsetext)
		gameOverGifUrl.value = response_objects['results'][randomGifIndex]['media'][0]['gif']['url']
	})
}

// url Async requesting function
function httpGetAsync(theUrl, callback) {
	// create the request object
	var xmlHttp = new XMLHttpRequest()

	// set the state change callback to capture when the response comes in
	xmlHttp.onreadystatechange = function () {
		if (xmlHttp.readyState == 4 && xmlHttp.status == 200) {
			callback(xmlHttp.responseText)
		}
	}

	// open as a GET call, pass in the url and set async = True
	xmlHttp.open('GET', theUrl, true)

	// call send with no params as they were passed in on the url string
	xmlHttp.send(null)

	return
}
</script>

<template>
	<div class="game" :class="{ gameRunning: !gameOver }">
		<div class="panel scoreboard">
			<div class="digitalDisplay minecount">
				{{ markedCellsCount.toString().padStart(3, '0') }}
			</div>
			<div class="digitalDisplay timer">
				{{ timer.toString().padStart(3, '0') }}
			</div>
		</div>
		<div class="panel minefield">
			<CellComp
				v-for="(cell, index) in cells"
				:index="index"
				:mined="cell.mined"
				:state="cell.state"
				:adjacentMinesCount="cell.adjacentMinesCount"
				@revealCellByIndex="revealCellByIndex"
				@markCellByIndex="markCellByIndex"
			/>
			<div v-if="gameOverGifUrl" class="gameOverResult">
				<p class="message">{{ gameOverMessage }}</p>
				<img :src="gameOverGifUrl" />
			</div>
		</div>
	</div>
</template>

<style scoped>
.game {
	margin: 0 auto;
	width: 60%;
	border: 0.3rem outset #bbb;
	background: var(--color-background-game);
	user-select: none;
}
.panel {
	border: 0.3rem inset #bbb;
}
.scoreboard {
	display: flex;
	justify-content: space-between;
}
.digitalDisplay {
	margin: 0.2rem;
	border: 0.2rem inset #bbb;
	padding: 0.1rem;
	color: #ff0000;
	background-color: #000;
	font-family: '7segments';
	font-size: 2rem;
	line-height: 1em;
	font-weight: bold;
	cursor: default;
}
.digitalDisplay::before {
	content: '888';
	position: absolute;
	color: #ff000058;
}
.minefield {
	position: relative;
	display: grid;
	grid-template-columns: repeat(16, 1fr);
	grid-template-rows: repeat(16, 1fr);
}
.cell {
	display: flex;
	justify-content: center;
	align-items: center;
	aspect-ratio: 1;
	color: #444;
	container-type: inline-size;
	font-size: 1cqi;
}
.cell.fresh,
.cell.marked {
	border: 0.3rem outset #bbb;
}
.cell.fresh {
	cursor: pointer;
}
.cell.fresh:hover:active {
	border: none;
}
.cell.mined {
	background: #ff4a4a;
}
.gameOverResult {
	position: absolute;
	width: 100%;
	height: 100%;
	display: flex;
	flex-direction: column;
	justify-content: space-evenly;
	background-color: #000000c2;
}
.gameOverResult .message {
	text-align: center;
	font-size: 2em;
	color: hsla(160, 100%, 37%, 1);
}
.gameOverResult img {
	width: 100%;
	max-height: calc(100% - 3em);
}

@media (min-width: 1024px) {
	.game {
		width: auto;
	}
	.minefield {
		margin: 0;
		width: auto;
	}
}
</style>
