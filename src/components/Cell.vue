<script setup>
import { ref } from 'vue'

const props = defineProps({
	index: {
		type: Number,
		required: true
	},
	mined: {
		type: Boolean,
		required: true
	},
	adjacentMinesCount: {
		type: Number,
		required: true
	},
	revealedInit: {
		type: Boolean,
		required: true
	}
})

let revealed = ref(props.revealedInit)

const onClick = () => !revealed.value && reveal()

const reveal = () => {
	revealed.value = true

	if (props.mined) {
		// @todo: end the game
		//  - reveal all mines
		//  - stop clickhandler
		//  - add restart button
	} else if (props.adjacentMinesCount == 0) {
		// @todo: propagate the reveal to all adjacent cells
	}
}
</script>

<template>
	<div
		class="cell"
		:class="{ unrevealed: !revealed, mined: revealed && mined }"
		@click.stop="onClick()"
	>
		<!-- <div class="index" v-if="!revealed">{{ index }}</div> -->
		<div class="adjacentMinesCount" v-if="revealed && !mined && adjacentMinesCount">
			{{ adjacentMinesCount }}
		</div>
	</div>
</template>

<style scoped>
.cell {
	display: flex;
	justify-content: center;
	align-items: center;
	aspect-ratio: 1;
	user-select: none;
	color: #444;
	background: #999;
	container-type: inline-size;
}
.cell .index {
	font-size: 35cqw;
}
.cell .adjacentMinesCount {
	font-size: 55cqw;
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
