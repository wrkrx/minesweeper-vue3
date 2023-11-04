<script setup>
import { config } from '@/stores/config.js'

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
	revealed: {
		type: Boolean,
		required: true
	}
})
const emit = defineEmits(['revealCellByIndex'])
</script>

<template>
	<div
		class="cell"
		:class="{ unrevealed: !revealed, mined: revealed && mined }"
		@click.stop="$emit('revealCellByIndex', index)"
	>
		<div class="index" v-if="config.debug && !revealed">{{ index }}</div>
		<div class="adjacentMinesCount" v-if="revealed && !mined && adjacentMinesCount">
			{{ adjacentMinesCount }}
		</div>
	</div>
</template>
