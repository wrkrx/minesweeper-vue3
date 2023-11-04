<script setup>
import Cell from './Cell.vue'
import { ref } from 'vue'

const rowsCount = 16
const colsCount = 16

const maxRow = rowsCount - 1
const maxCol = colsCount - 1

const cellsCount = rowsCount * colsCount
const maxCell = cellsCount - 1

const minesCount = 10

const getIndexFromCoords = (column, row) => {
	return rowsCount * row + column
}

const getAdjacentCellsIndices = (index) => {
	let result = []

	let column = index % rowsCount
	let row = Math.floor(index / rowsCount)

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
for (var i = maxCell; i >= 0; i--) {
	cellsInit[i] = {
		mined: false,
		revealed: false,
		adjacentMinesCount: 0
	}
}

// place mines by shuffling cells and selecting x first cells
// @todo: implement a better randomisation of the cells
const shuffledCells = cellsInit.toSorted(() => 0.5 - Math.random())
let selectedCells = shuffledCells.slice(0, minesCount)
for (var i = selectedCells.length - 1; i >= 0; i--) {
	selectedCells[i].mined = true
}

// compute adjacent mines number for each cells
for (var i = maxCell; i >= 0; i--) {
	const adjacentIndices = getAdjacentCellsIndices(i)
	for (var j = adjacentIndices.length - 1; j >= 0; j--) {
		let adjacentIndex = adjacentIndices[j]
		if (adjacentIndex >= 0 && adjacentIndex < cellsCount) {
			if (cellsInit[adjacentIndex].mined) {
				cellsInit[i].adjacentMinesCount++
			}
		}
	}
}

const cells = ref(cellsInit)

const onReveal = (cellIndex) => {
	const cell = cells.value[cellIndex]

	if (cell.revealed) return

	cell.revealed = true

	if (cell.mined) {
		// @todo: end the game
		//  - reveal all mines
		//  - stop clickhandler
		//  - add restart button
	} else if (cell.adjacentMinesCount == 0) {
		const adjacentIndices = getAdjacentCellsIndices(cellIndex)
		for (var i = adjacentIndices.length - 1; i >= 0; i--) {
			const adjacentIndex = adjacentIndices[i]
			onReveal(adjacentIndex)
		}
	}
}
</script>

<template>
	<div class="minefield">
		<Cell
			v-for="(cell, index) in cells"
			:index="index"
			:mined="cell.mined"
			:revealed="cell.revealed"
			:adjacentMinesCount="cell.adjacentMinesCount"
			@reveal="onReveal"
		/>
	</div>
</template>

<style>
.minefield {
	margin: 0 auto;
	width: 60%;
	display: grid;
	grid-template-columns: repeat(16, 1fr);
	grid-template-rows: repeat(16, 1fr);
}
@media (min-width: 1024px) {
	.minefield {
		margin: 0;
		width: auto;
	}
}
.cell {
	display: flex;
	justify-content: center;
	align-items: center;
	aspect-ratio: 1;
	user-select: none;
	color: #444;
	background: #999;
	container-type: inline-size;
	font-size: 1.5cqi;
}
.cell .index {
	font-size: 35cqi;
}
.cell.unrevealed {
	border: 0.3rem outset #bbb;
	cursor: pointer;
}
.cell.unrevealed:hover:active {
	border: none;
}
.cell.mined {
	background: #ff4a4a;
}
.cell.mined::before {
	content: '⬤';
}
</style>
