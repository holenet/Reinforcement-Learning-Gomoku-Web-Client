<script>
	import { createEventDispatcher } from 'svelte';

	export let color;
	export let board;
	export let moves;
	export let lastMove;
	export let isMyTurn;

	const dispatch = createEventDispatcher();

	const size = 15;
	const unit = 1 / size;
	const lineWidth = unit / size;
	let selectedCell = {
		i: null,
		j: null
	};

	function handleSelectCell(i, j) {
		selectedCell = { i, j };
	}
	function handleMove(i, j) {
		if (board[i][j] == 0) {
			dispatch('move', { i, j });
			selectedCell = {
				i: null,
				j: null
			};
		}
	}
</script>

{#if color !== null}
	<div class="color-status">
		You are {color == 1 ? 'BLACK' : color == -1 ? 'WHITE' : 'SPECTATOR'}
	</div>
{/if}
<svg viewBox="0 0 1 1">
	<rect x={0} y={0} width={1} height={1} fill="#FFCCAA" />
	{#each { length: size } as _, i}
		<line
			x1={unit * 0.5}
			y1={unit * (i + 0.5)}
			x2={1 - unit * 0.5}
			y2={unit * (i + 0.5)}
			stroke="black"
			stroke-width={lineWidth}
		/>
	{/each}
	{#each { length: size } as _, i}
		<line
			y1={unit * 0.5}
			x1={unit * (i + 0.5)}
			y2={1 - unit * 0.5}
			x2={unit * (i + 0.5)}
			stroke="black"
			stroke-width={lineWidth}
		/>
	{/each}
	{#each moves as move, index}
		{#if lastMove && move.i == lastMove.i && move.j == lastMove.j}
			<rect
				x={unit * move.i}
				y={unit * move.j}
				width={unit}
				height={unit}
				rx={lineWidth * 2}
				ry={lineWidth * 2}
				fill="#FF555599"
			/>
		{/if}
		<circle
			r={unit * 0.4}
			cx={unit * (move.i + 0.5)}
			cy={unit * (move.j + 0.5)}
			fill={move.color == -1 ? 'white' : 'black'}
			stroke="black"
			stroke-width={lineWidth * 0.5}
		/>
		<text
			x={unit * (move.i + 0.5)}
			y={unit * (move.j + 0.5) + unit * 0.06}
			fill={move.color == -1 ? 'black' : 'white'}
			font-family="Leckerli One"
			font-size={unit * 0.6}
			textLength={unit * 0.6}
			lengthAdjust="spacing"
			font-weight="700"
			dominant-baseline="middle"
			text-anchor="middle"
		>
			{index + 1}
		</text>
	{/each}
	{#each board as row, i}
		{#each row as _, j}
			<rect
				x={unit * i}
				y={unit * j}
				width={unit}
				height={unit}
				fill={isMyTurn && i == selectedCell.i && j == selectedCell.j && board[i][j] == 0
					? '#44AAFF66'
					: 'transparent'}
				rx={lineWidth * 2}
				ry={lineWidth * 2}
				focusable="false"
				on:focus|preventDefault={() => handleSelectCell(i, j)}
				on:mouseover|preventDefault={() => handleSelectCell(i, j)}
				on:click|preventDefault={() => (isMyTurn ? handleMove(i, j) : '')}
			/>
		{/each}
	{/each}
</svg>

<style>
	.color-status {
		padding-bottom: 8px;
	}

	svg {
		width: 100%;
		height: 100%;
	}

	svg rect,
	svg rect:focus {
		outline: none;
	}
</style>
