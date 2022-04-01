<script>
	import { createEventDispatcher } from 'svelte';

	export let arenas;

	const dispatch = createEventDispatcher();

	function handleEnterArena(arena) {
		dispatch('enter', arena);
	}
	function handleSpectateArena(arena) {
		dispatch('spectate', arena);
	}
</script>

<div class="lobby">
	<div class="lobby-arena-list">
		{#each arenas as arena}
			<div class="lobby-arena-item">
				<div>
					<div class="lobby-arena-item__title">
						[{arena.players === 2 ? 'PVP' : arena.players === 1 ? 'PVAI' : 'AIVAI'}] {arena.title}
					</div>
					<div class="lobby-arena-item__info">
						{arena.agents} agents, {arena.spectators} spectators
					</div>
					<div class="lobby-arena-item__id">
						{arena.id}
					</div>
				</div>
				<div class="lobby-arena-item__buttons">
					{#if arena.agents < 2}
						<button on:click|preventDefault={() => handleEnterArena(arena)}>ENTER</button>
					{/if}
					<button on:click|preventDefault={() => handleSpectateArena(arena)}>SPECTATE</button>
				</div>
			</div>
		{/each}
	</div>
</div>

<style lang="scss">
	.lobby {
		height: 100%;
		overflow-y: auto;

		&-header {
			width: 100%;
			display: flex;
			flex-direction: row-reverse;
		}

		&-arena-list {
			display: flex;
			flex-direction: column;
			gap: 16px;
		}

		&-arena-item {
			display: flex;
			justify-content: space-between;
			align-items: center;

			&__title {
				font-size: 1.5em;
				font-weight: 700;
			}

			&__info {
				color: gray;
			}

			&__id {
				font-size: 0.5em;
				color: gray;
			}

			&__buttons {
				display: flex;
				flex-direction: column;
				justify-content: center;
				align-items: flex-end;
				gap: 4px;
			}
		}
	}
</style>
