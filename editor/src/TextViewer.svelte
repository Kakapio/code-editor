<!-- TextEditor.svelte -->
<script>
	import { writable } from 'svelte/store';

	export let text = '';

	/**
	 * @type {{ [x: string]: any; }[]}
	 */
	export let rewrites = [];
	export let originalText = '';
	/**
	 * @type {{value: string; flag: boolean}[]}
	 * Each index represents the state of the diff after the nth rewrite is applied.
	 */
	let beforeFrames = [];
	/**
	 * @type {{value: string; flag: boolean}[]}
	 */
	let afterFrames = [];

	/**
	 * @type {{ [x: string]: any; }[]}
	 * These are the finished sets of rows for each step. Essentially rendered data.
	 * Each steps changes are applied to the output of the previous step.
	 * This means we can get the state at each point even when recursive rewrites are applied.
	 */
	let activeTab = 'textarea';
	let currentStep = writable(0);

	/**
	 * @param {{ target: { value: string; }; }} event
	 */
	export function handleChange(event) {
		text = event.target.value;
	}

	/**
	 * Create all our frames and properly organize the output data.
	 */
	export function createDiff() {
		let initialText = originalText;
		beforeFrames.length = 0; // Clear these out.
		afterFrames.length = 0;
		currentStep = writable(0);

		for (let i = 0; i < rewrites.length; i++) {
			let out = generateFrame(initialText, i);

			// Create our tables here.
			let beforeLines = initialText.split("\n");
			let beforeTuples = beforeLines.map((line, index) => {
				if (index >= out.startRow && index <= out.endRow) {
					return { value: line, flag: true };
				}
				else {
					return { value: line, flag: false };
				}
			});

			let afterLines = out.after.split("\n");
			let afterTuples = afterLines.map((line, index) => {
				if (index >= out.startRow && index <= out.endRow) {
					return { value: line, flag: true };
				}
				else {
					return { value: line, flag: false };
				}
			});

			initialText = out.after; // Setup for next rewrite...

			beforeFrames.push(beforeTuples);
			afterFrames.push(afterTuples);
		}
	}

	export function clear() {
		beforeFrames.length = 0; // Clear our tables.
		afterFrames.length = 0;
	}

	/**
	 * @param {string} tabName
	 */
	function switchTab(tabName) {
		activeTab = tabName;
	}

	// Function to increment currentStep
	function addStep() {
		currentStep.update((value) => {
			if (value < rewrites.length - 1) {
				return value + 1;
			}
			return value;
		});
	}

	function subtractStep() {
		currentStep.update((value) => {
			if (value > 0) {
				return value - 1;
			}
			return value;
		});
	}

	/**
	 * @param {string} [inputText] Generate a frame using a given rewrite.
	 * @param {number} [index]
	 */
	function generateFrame(inputText, index) {
		let rewrite = rewrites[index];
		let afterText = inputText.slice(0, rewrite.range.start_byte) + rewrite.replacement + inputText.slice(rewrite.range.end_byte);
		return { after: afterText, startRow: rewrite.range.start_point.row, endRow: rewrite.range.end_point.row };
	}
</script>

<div class="container">
	<div class="tab-buttons">
		<button on:click={() => switchTab('textarea')} class:selected={activeTab === 'textarea'}
			>Normal</button
		>
		<button on:click={() => switchTab('table')} class:selected={activeTab === 'table'}
			>Rewrite Stepper</button
		>
		{#if rewrites.length > 0 && activeTab === 'table'}
			<button on:click={subtractStep}>Step Back</button>
			<button on:click={addStep}>Step Forward</button>
			<button disabled>{($currentStep + 1) + "/" + rewrites.length}</button>
		{/if}
	</div>
	<div class="tab-content">
		{#if activeTab === 'textarea'}
			<textarea
				bind:value={text}
				readonly={true}
				class:default-text={text === 'This is where your cleaned up code will be...'}
			></textarea>
		{:else if activeTab === 'table' && beforeFrames.length > 0}
			<div class="two-column-table">
				<table>
					<thead>
						<tr>
							<th>Original</th>
						</tr>
					</thead>
					<tbody>
						{#each beforeFrames[$currentStep] as { value, flag }}
							<tr>
								<td><pre class:codered={flag}>{value}</pre></td>
							</tr>
						{/each}
					</tbody>
				</table>

				<table>
					<thead>
						<tr>
							<th>Replacement</th>
						</tr>
					</thead>
					<tbody>
						{#each afterFrames[$currentStep] as { value, flag }}
							<tr>
								<td><pre class:codegreen={flag}>{value}</pre></td>
							</tr>
						{/each}
					</tbody>
				</table>
			</div>
		{/if}
	</div>
</div>

<style>
	textarea {
		width: 100%;
		height: 500px;
		border: 2px solid #21242d;
		padding: 10px;
		font-size: 16px;
		background-color: #191b22;
		color: white;
	}

	td {
		color: white;
		font-family: 'Roboto', sans-serif;
	}

	th {
		color: rgb(95, 218, 255);
		font-family: 'Roboto', sans-serif;
	}

	tr {
		background-color: #191b22;
		line-height: 0.7; /* Adjust this value as needed */
	}

	table {
		width: 100%;
		height: 523px;
		border-spacing: 0; /* Remove gaps between table rows */
		border: 2px solid #21242d;
	}

	.codered {
		line-height: 0.1; /* Adjust this value as needed */
		background-color: #642121;
	}

	.codegreen {
		line-height: 0.1; /* Adjust this value as needed */
		background-color: #246421;
	}

	button {
		font-size: 16px;
		font-family: 'Roboto', sans-serif;
		background-color: #21242d;
		color: rgb(169, 169, 169);
		border-radius: 0px;
	}

	.default-text {
		font-style: italic;
		color: rgb(169, 169, 169);
	}

	.selected {
		font-weight: bold;
		color: rgb(95, 218, 255);
	}

	.container {
		display: flex;
		flex-direction: column;
		width: 55%;
		margin-top: -25px;
	}

	.tab-buttons {
		display: flex;
		justify-content: start;
	}

	.tab-content {
		flex-grow: 1;
	}

	.two-column-table {
		display: flex;
	}

	/* Define custom scrollbar colors for entire site */
	:global(html) {
		scrollbar-color: #21242d #191b22; /* thumb color, track color */
	}
</style>
