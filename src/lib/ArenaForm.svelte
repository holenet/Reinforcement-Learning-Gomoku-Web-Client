<script>
	import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	let title = '';
	let players = 2;
	let spectator = true;
	const modeOptions = [
		{ value: 2, name: 'PVP' },
		{ value: 1, name: 'PVAI' },
		{ value: 0, name: 'AIVAI' }
	];

	function handleCreateArena() {
		dispatch('submit', {
			title,
			players,
			spectator
		});
	}
</script>

<div class="form">
	<div class="form-item">
		<div class="form-item__label">Title</div>
		<div class="form-item__input">
			<input type="text" bind:value={title} placeholder="Arena" />
		</div>
	</div>
	<div class="form-item">
		<div class="form-item__label">Mode</div>
		<div class="form-item__input">
			<select bind:value={players}>
				{#each modeOptions as { value, name }}
					<option {value}>
						{name}
					</option>
				{/each}
			</select>
		</div>
	</div>
	<!-- <div class="form-item">
		<div class="form-item__label">Spectator</div>
		<div class="form-item__input">
			<input type="checkbox" checked={spectator} />
		</div>
	</div> -->
	<button class="form__submit-button" on:click={handleCreateArena}>CREATE</button>
</div>

<style lang="scss">
	.form {
		width: 100%;
		height: 100%;
		display: flex;
		flex-direction: column;
		gap: 8px;

		&-item {
			display: flex;

			&__label {
				font-weight: 700;
				width: 40%;
			}
			&__input {
				width: 60%;
			}
		}

		&__submit-button {
			margin-top: auto;
		}
	}
</style>
