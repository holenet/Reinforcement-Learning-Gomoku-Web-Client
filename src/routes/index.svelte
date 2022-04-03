<script>
	import '../app.css';
	import Board from '$lib/Board.svelte';
	import Lobby from '$lib/Lobby.svelte';
	import ArenaForm from '$lib/ArenaForm.svelte';
	import { onMount } from 'svelte';

	const initialArenaState = () => ({
		id: null,
		title: '',
		isGameStarted: false,
		agents: 0,
		spectators: 0
	});
	const initialGameState = () => ({
		winner: null,
		lastMove: null,
		isGameOver: false,
		nextTurn: null,
		isMyTurn: null,
		moves: [],
		board: []
	});

	let isCreatingArena = false;
	let arenaList = [];
	let color = null;
	let arenaState = initialArenaState();
	let gameState = initialGameState();
	let connected = null;
	let log = '';

	let logArea;
	let ws;

	$: gameStatus = gameState.isGameOver
		? gameState.winner === color
			? 'You win!'
			: gameState.winner === -color
			? 'You lose!'
			: gameState.winner === null
			? 'Draw!'
			: `${gameState.winner === 1 ? 'BLACK' : 'WHITE'} win!`
		: gameState.isMyTurn
		? 'Your turn'
		: color === 0
		? `${gameState.nextTurn === 1 ? 'BLACK' : 'WHITE'} turn`
		: 'Waiting opponent...';
	$: connectionStatus =
		connected === null ? 'Connecting...' : connected ? 'Connected' : 'Disconnected';

	function reset() {
		isCreatingArena = false;
		arenaList = [];
		color = null;
		arenaState = initialArenaState();
		gameState = initialGameState();
		connected = null;
		log = '';

		if (ws) ws.close();
		ws = new WebSocket('ws://dev.holenet.net:5000');
		ws.onopen = (m) => (connected = true);
		ws.onclose = (m) => {
			connected = false;
		};
		ws.onerror = (m) => alert('An error occurred.');
		ws.onmessage = (m) => {
			const message = JSON.parse(m.data);
			if (message.type === 'ARENA_LIST') {
				arenaList = message.data.arenas;
			} else if (message.type === 'ARENA_STATE') {
				arenaState = {
					...message.data
				};
				isCreatingArena = false;
			} else if (message.type === 'START_GAME') {
				color = message.data.color;
				isCreatingArena = false;
			} else if (message.type === 'GAME_STATE' || message.type === 'REQUEST_MOVE') {
				gameState = {
					...message.data,
					isMyTurn: message.data.nextTurn === color
				};
			}
			log += `[${getCurrentTime()}] ${message.type}: ${
				message.message ?? JSON.stringify(message.data).substring(0, 50) + '...'
			}\n`;
			setTimeout(() => (logArea.scrollTop = logArea.scrollHeight), 1);
		};
	}

	function getCurrentTime() {
		const now = new Date();
		return `${now.getHours()}:${now.getMinutes()}:${now.getSeconds()}`;
	}

	function sendMessage(type, message = null) {
		if (!connected || !ws) {
			window.location.reload();
			return;
		}
		ws.send(
			JSON.stringify({
				type,
				data: message
			})
		);
	}

	function close() {
		if (ws) ws.close();
	}

	function move(event) {
		const position = event.detail;
		sendMessage('MOVE', position);
	}

	function pass() {
		sendMessage('PASS');
	}

	function createArena(event) {
		const data = event.detail;
		sendMessage('CREATE_ARENA', data);
	}

	function enterArena(event) {
		const arena = event.detail;
		sendMessage('ENTER_ARENA', {
			id: arena.id
		});
	}

	function spectateArena(event) {
		const arena = event.detail;
		sendMessage('SPECTATE_ARENA', {
			id: arena.id
		});
	}

	function showCreateArena() {
		isCreatingArena = true;
	}

	function hideCreateArena() {
		isCreatingArena = false;
	}

	onMount(reset);

	onbeforeunload = function () {
		if (ws) ws.close();
	};
</script>

<div class="header">
	<div>
		{#if arenaState.title}
			<div class="header__title">
				[{arenaState.players === 2 ? 'PVP' : arenaState.players === 1 ? 'PVAI' : 'AIVAI'}] {arenaState.title}
			</div>
		{/if}
	</div>
	<div>
		Server {connectionStatus}
		{#if connected === false || arenaState.id === null}
			<button on:click|preventDefault={reset}>RESET</button>
		{/if}
	</div>
</div>
<div class="container">
	<div class="split-item">
		{#if arenaState.id !== null}
			<Board {color} {...gameState} on:move={move} />
		{:else if isCreatingArena}
			<ArenaForm on:submit={createArena} />
		{:else}
			<Lobby arenas={arenaList} on:enter={enterArena} on:spectate={spectateArena} />
		{/if}
	</div>
	<div class="split-item">
		<div class="info">
			<div class="info-header">
				<div class="info-header__text">
					{#if arenaState.id !== null}
						<div>{arenaState.agents} agents, {arenaState.spectators} spectators</div>
						<div>{gameStatus}</div>
					{/if}
				</div>
				<div class="info-header__buttons">
					{#if arenaState.id === null && !isCreatingArena}
						<button on:click|preventDefault={showCreateArena}>CREATE ARENA</button>
					{/if}
					{#if isCreatingArena}
						<button on:click|preventDefault={hideCreateArena}>CLOSE</button>
					{/if}
					{#if gameState.isMyTurn}
						<button on:click|preventDefault={pass}>PASS</button>
					{/if}
					{#if arenaState.id !== null}
						<button on:click|preventDefault={reset}>EXIT</button>
					{/if}
				</div>
			</div>
			<textarea class="info-log" bind:this={logArea} disabled>{log}</textarea>
		</div>
	</div>
</div>

<style lang="scss">
	@media (max-width: 600px) {
		.container {
			flex-direction: column;
		}
	}

	@media (min-width: 600px) {
		.container {
			flex-direction: row;
		}
	}

	.header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 16px;
		gap: 8px;
		color: white;
		background-color: black;

		&__title {
			font-size: 1.5em;
			font-weight: 700;
		}
	}

	.container {
		display: flex;
		gap: 16px;
		padding: 16px;
		height: 100%;
	}

	.split-item {
		flex: 1 1 0;
	}

	.info {
		display: flex;
		flex-direction: column;
		gap: 16px;
		width: 100%;
		height: 100%;

		&-header {
			display: flex;
			flex-direction: row;
			gap: 16px;
			justify-content: space-between;
			align-items: center;

			&__text {
				display: flex;
				flex-direction: row-reverse;
				justify-content: space-between;
				align-items: center;
				flex: 1 1 auto;
			}
		}

		&-log {
			width: 100%;
			flex: 1 0 0;
			min-height: 200px;
		}
	}
</style>
