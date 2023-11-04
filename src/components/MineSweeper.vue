<script setup>
import Cell from './Cell.vue'
import { ref } from 'vue'
import { cellState } from '@/stores/enums.js'

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
// @todo: implement a better randomisation of the cells
const shuffledCells = cellsInit.toSorted(() => 0.5 - Math.random())
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

const revealCellByIndex = (cellIndex) => {
	const cell = cells.value[cellIndex]

	if (cell.state == cellState.revealed) return
	if (cell.state == cellState.marked) return

	cell.state = cellState.revealed

	if (cell.mined) {
		// @todo: end the game
		//  - reveal all mines
		//  - stop clickhandler
		//  - add restart button
	} else if (cell.adjacentMinesCount == 0) {
		const adjacentIndices = getAdjacentCellsIndices(cellIndex)
		for (let i = adjacentIndices.length - 1; i >= 0; i--) {
			const adjacentIndex = adjacentIndices[i]
			revealCellByIndex(adjacentIndex)
		}
	}
}
const markCellByIndex = (cellIndex) => {
	const cell = cells.value[cellIndex]

	if (cell.state == cellState.revealed) return

	if (cell.state == cellState.marked) {
		cell.state = cellState.fresh
	} else {
		cell.state = cellState.marked
	}
}
</script>

<template>
	<div class="minefield">
		<Cell
			v-for="(cell, index) in cells"
			:index="index"
			:mined="cell.mined"
			:state="cell.state"
			:adjacentMinesCount="cell.adjacentMinesCount"
			@revealCellByIndex="revealCellByIndex"
			@markCellByIndex="markCellByIndex"
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
	border: 0.6rem ridge #bbb;
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
.cell.fresh {
	cursor: pointer;
}
.cell.fresh,
.cell.marked {
	border: 0.3rem outset #bbb;
}
.cell.fresh:hover:active {
	border: none;
}
.cell.marked::before {
	content: '🚩';
}
.cell.mined {
	background: #ff4a4a;
}
.cell.mined::before {
	content: '⬤';
}
</style>
