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
	 * @type {string[]}
	 * The original data in a list of lines.
	 */
	let originalLines = [];
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
	let renderedRewrites = [];
	let activeTab = 'textarea';
	let currentStep = writable(0);

	/**
	 * @param {{ target: { value: string; }; }} event
	 */
	export function handleChange(event) {
		text = event.target.value;
	}

	/**
	 * This is only run when we successfully generate cleaned up code.
	 */
	export function createDiff() {
		originalLines = originalText.split('\n');
		// Map over originalLines and transform each line into a tuple
		let tuples = originalLines.map((line) => ({ value: line, flag: true }));

		// Until all our rewrites are consumed or we hit 10 rewrites.
		let timeoutLimit = 0;
		while (rewrites.length > 0 && timeoutLimit < 10) {
			if (afterFrames.length == 0) {
				// First iteration
				let out = generateFrame(tuples);
				beforeFrames.push(out.before); // out.before is the beforeList of tuples <string, bool>.
				afterFrames.push(out.after);
			} else {
				console.warn(
					'Beforeframes has ' +
						beforeFrames.length +
						'Afterframes has ' +
						(afterFrames.length - 1) +
						' in it'
				);
				let out = generateFrame(afterFrames[afterFrames.length - 1]); // Use our newest frame for the next frame.
				beforeFrames.push(out.before);
				afterFrames.push(out.after);
			}

			timeoutLimit++;
		}
	}

	export function clear() {
		text = 'This is where your cleaned up code will be...';
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
	 * @param {{value: string, flag: boolean}[]} [inputLines]
	 */
	function generateFrame(inputLines) {
		inputLines = inputLines || []; // Provide a default value if inputLines is undefined
		let beforeLines = []; // Tables to hold this particular frame.
		let afterLines = [];

		let currBytes = 0; // the number of bytes we have consumed thus far. Used to track our position.
		let foundMatchEver = false; // whether we have found any matches during this function call.
		console.error('Rewrites length is ' + rewrites.length);

		// I make a lot of assumptions here about rewrites being by line, etc...
		// A rewrite occurring in the middle of a line would break this. But even GitHub doesn't do that.
		for (let i = 0; i < inputLines.length; i++) {
			let line = inputLines[i].value;
			console.warn('This should be the same line: ' + JSON.stringify(inputLines[i].value));
			// Count how much white space we have... This is ignored on the backend so we must also do that.
			let whitespaceCount = line.length - line.trimStart().length;
			let foundLineMatch = false; // So we can track whether a match was found for this particular line.

			// Check the bounds of each line against every item in the rewrite array...
			// This would be more efficient if rewrite array was ordered by start_byte. Could do O(n) rather than (n^2).
			for (let j = 0; j < rewrites.length; j++) {
				let rewrite = rewrites[j];

				let start = currBytes; // Starting byte of this line.
				let end = start + line.length; // End byte.

				// console.log(
				// 	'line:' +
				// 		line +
				// 		'\nstart: ' +
				// 		start +
				// 		' start_byte: ' +
				// 		rewrite.range.start_byte +
				// 		'\nend: ' +
				// 		end +
				// 		' end_byte: ' +
				// 		rewrite.range.end_byte +
				// 		'\n first condition: ' +
				// 		(rewrite.range.start_byte >= start) +
				// 		' second condition: ' +
				// 		(rewrite.range.end_byte <= end) +
				// 		'\ncurrBytes: ' +
				// 		currBytes +
				// 		' whitespacecount: ' +
				// 		whitespaceCount
				// );

				// This particular line has been found in our rewrites...
				// !foundMatchEver is here because once we find a match for a rewrite, we are done. We limit
				// each diff to one change.
				if (!foundMatchEver && rewrite.range.start_byte >= start && rewrite.range.end_byte <= end) {
					console.log("Found a diff at line: " + i + "\npushing before: " + JSON.stringify(rewrite.original)
					+ "\npushing after: " + JSON.stringify(rewrite.replacement));
					rewrites = rewrites.filter((_, index) => index !== j); // Remove this used rewrite
					beforeLines.push({ value: rewrite.original, flag: true });
					afterLines.push({ value: rewrite.replacement, flag: true });
					foundLineMatch = true;
					foundMatchEver = true;
					break; // TODO: Could there be more than one rewrite for a given set of bytes? here we just assume not.
				}
			}

			// If we couldn't find a changed line, just use the original.
			if (!foundLineMatch) {
				//console.warn('Found nothing at line: ', i);
				beforeLines.push({ value: inputLines[i].value, flag: false });
				afterLines.push({ value: inputLines[i].value, flag: false });
			}

			currBytes += inputLines[i].value.length + 1; // Plus 1 for the new line we just traversed.
		}

		return { before: beforeLines, after: afterLines };
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
