<script setup>
import { cellState } from '@/stores/enums.js'

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
	state: {
		type: Number,
		required: true
	}
})
const emit = defineEmits(['revealCellByIndex', 'markCellByIndex'])

const onClick = () => {
	emit('revealCellByIndex', props.index)
}
const onRightClick = () => {
	emit('markCellByIndex', props.index)
}
</script>

<template>
	<div
		class="cell"
		:class="{
			fresh: state == cellState.fresh,
			marked: state == cellState.marked,
			mined: state == cellState.revealed && mined
		}"
		@click.stop.prevent="onClick"
		@contextmenu.stop.prevent="onRightClick"
	>
		<div v-if="state == cellState.marked">🚩</div>
		<div v-if="state == cellState.revealed && mined">⬤</div>
		<div
			class="adjacentMinesCount"
			v-if="state == cellState.revealed && !mined && adjacentMinesCount"
		>
			{{ adjacentMinesCount }}
		</div>
	</div>
</template>
