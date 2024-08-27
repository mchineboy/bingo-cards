<script lang="ts">
	import { onMount } from 'svelte';

	type BingoSquare = {
		text: string;
		checked: boolean;
	};

	let card: BingoSquare[] = [];
	let phrases: string[] = [];
	const MIDDLE_INDEX = 12; // Index of the middle square in a 5x5 grid

	async function loadPhrases(): Promise<void> {
		try {
			const response = await fetch('/maga_phrases.txt');
			const text = await response.text();
			phrases = text.split('\n').filter((phrase) => phrase.trim() !== '');
		} catch (error) {
			console.error('Failed to load phrases:', error);
			phrases = ['Failed to load phrases'];
		}
	}

	function shuffleArray(array: string[]): string[] {
		for (let i = array.length - 1; i > 0; i--) {
			const j = Math.floor(Math.random() * (i + 1));
			[array[i], array[j]] = [array[j], array[i]];
		}
		return array;
	}

	function generateCard(): void {
		if (phrases.length < 24) {
			console.error('Not enough phrases to generate a full card');
			return;
		}
		const shuffledPhrases = shuffleArray([...phrases]);

		card = shuffledPhrases.slice(0, 24).map((text) => ({ text, checked: false }));
		// Insert the FREE space in the middle
		card.splice(MIDDLE_INDEX, 0, { text: 'FREE SPACE', checked: false });
	}

	function toggleSquare(index: number): void {
		card[index].checked = !card[index].checked;
		card = [...card]; // Trigger reactivity
	}

	onMount(async () => {
		await loadPhrases();
		generateCard();
	});
</script>

<main>
	<h1 class="qwitcher-grypen-bold big-text">Live Debate</h1>
	<h3 class="rubik-mono-one-regular large-text red-text">BINGO</h3>
	{#if card.length === 25}
		<div class="bingo-card">
			{#each card as square, i}
				<div
					class="bingo-square"
					class:free={square.text === 'FREE SPACE'}
					class:checked={square.checked}
					on:click={() => toggleSquare(i)}
				>
					<span>{square.text}</span>
				</div>
			{/each}
		</div>
		<button on:click={generateCard}>New Bingo Card</button>
	{:else}
		<p>Loading phrases or not enough phrases available...</p>
	{/if}
</main>

<style>
	:root {
		--square-size: min(16vw, 100px);
	}

	main {
		text-align: center;
		font-family: Arial, sans-serif;
		max-width: 100vw;
		overflow-x: hidden;
	}

	h1 {
		font-size: clamp(1.5rem, 5vw, 2.5rem);
		margin-bottom: 1rem;
	}

	.bingo-card {
		display: grid;
		grid-template-columns: repeat(5, 1fr);
		gap: 3px;
		width: calc(var(--square-size) * 5 + 10px);
		margin: 10px auto;
	}

	.bingo-square {
		width: var(--square-size);
		height: var(--square-size);
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: clamp(0.6rem, 2vw, 0.9rem);
		background-color: #f0f0f0;
		border: 1px solid #ccc;
		padding: 2px;
		cursor: pointer;
		transition: background-color 0.3s;
		overflow: hidden;
	}

	.bingo-square span {
		display: -webkit-box;
		-webkit-line-clamp: 4;
		-webkit-box-orient: vertical;
		overflow: hidden;
		text-overflow: ellipsis;
		max-height: 100%;
	}

	.free {
		background-color: #ffd700;
		font-weight: bold;
	}

	.checked {
		background-color: #90ee90;
		text-decoration: line-through;
	}

	button {
		font-size: clamp(0.9rem, 3vw, 1.2rem);
		padding: 10px 20px;
		cursor: pointer;
		margin-top: 10px;
	}

	@media (max-width: 480px) {
		:root {
			--square-size: 16.5vw;
		}

		.bingo-card {
			width: 95vw;
		}

		.bingo-square {
			font-size: 0.75rem;
		}
	}
	.qwitcher-grypen-regular {
		font-family: 'Qwitcher Grypen', cursive;
		font-weight: 400;
		font-style: normal;
	}

	.qwitcher-grypen-bold {
		font-family: 'Qwitcher Grypen', cursive;
		font-weight: 700;
		font-style: normal;
	}

	.red-text {
		color: red;
	}

	.large-text {
		font-size: 2em;
		leading: 0.1em;
		margin-top: 0px;
		margin-bottom: 0px;
	}

	.big-text {
		font-size: 7em;
		leading: 0.1em;
		margin-top: 0px;
		margin-bottom: 0px;
	}

	.rubik-mono-one-regular {
		font-family: 'Rubik Mono One', monospace;
		font-weight: 400;
		font-style: normal;
	}
</style>
