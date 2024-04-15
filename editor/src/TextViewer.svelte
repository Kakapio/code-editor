<!-- TextEditor.svelte -->
<script>
	export let text = '';

	/**
	 * @type {{ [x: string]: any; }[]}
	 */
	export let rewrites = [];

	let activeTab = 'textarea';
	export let originalText = '';
	/**
	 * @type {string[]}
	 * The original data in a list of lines.
	 */
	let originalLines = [];
	/**
	 * @type {{value: string; flag: boolean}[]}
	 */
	let beforeLines = [];
	/**
	 * @type {{value: string; flag: boolean}[]}
	 */
	let afterLines = [];

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
		generateTables();
	}

	export function clear() {
		text = 'This is where your cleaned up code will be...';
		beforeLines.length = 0; // Clear our tables.
		afterLines.length = 0;
	}

	/**
	 * @param {string} tabName
	 */
	function switchTab(tabName) {
		activeTab = tabName;
	}

	function generateTables() {
		beforeLines.length = 0; // Clear our tables.
		afterLines.length = 0;

		let currBytes = 0; // the number of bytes we have consumed thus far. Used to track our position.
		console.log('Rewrites length is ' + rewrites.length);

		// I make a lot of assumptions here about rewrites being by line, etc...
		// A rewrite occurring in the middle of a line would break this. But even GitHub doesn't do that.
		for (let i = 0; i < originalLines.length; i++) {
			let line = originalLines[i];
			// Count how much white space we have... This is ignored on the backend so we must also do that.
			let whitespaceCount = line.length - line.trimStart().length;
			let foundMatch = false; // So we can track whether a match was found outside the inner loop.

			// Check the bounds of each line against every item in the rewrite array...
			// This would be more efficient if rewrite array was ordered by start_byte. Could do O(n) rather than (n^2).
			for (let j = 0; j < rewrites.length; j++) {
				let rewrite = rewrites[j];

				let start = currBytes; // Starting byte of this line.
				let end = start + line.length; // End byte.

				console.log(
					'line:' +
						line +
						'\nstart: ' +
						start +
						' start_byte: ' +
						rewrite.range.start_byte +
						'\nend: ' +
						end +
						' end_byte: ' +
						rewrite.range.end_byte +
						'\n first condition: ' +
						(rewrite.range.start_byte >= start) +
						' second condition: ' +
						(rewrite.range.end_byte <= end) +
						'\ncurrBytes: ' +
						currBytes +
						' whitespacecount: ' +
						whitespaceCount
				);

				// This particular line has been found in our rewrites...
				if (rewrite.range.start_byte >= start && rewrite.range.end_byte <= end) {
					console.warn('Found a diff at line: ', i);
					beforeLines.push({ value: rewrite.original, flag: true });
					afterLines.push({ value: rewrite.replacement, flag: true });
					foundMatch = true;
					break; // TODO: Could there be more than one rewrite for a given set of bytes? here we just assume not.
				}
			}

			if (!foundMatch) {
				console.warn('Found nothing at line: ', i);
				beforeLines.push({ value: originalLines[i], flag: false });
				afterLines.push({ value: originalLines[i], flag: false });
			}

			currBytes += originalLines[i].length + 1; // Plus 1 for the new line we just traversed.
		}
	}
</script>

<div class="container">
	<div class="tab-buttons">
		<button on:click={() => switchTab('textarea')} class:selected={activeTab === 'textarea'}
			>Normal</button
		>
		<button on:click={() => switchTab('table')} class:selected={activeTab === 'table'}>Diff</button>
	</div>
	<div class="tab-content">
		{#if activeTab === 'textarea'}
			<textarea
				bind:value={text}
				readonly={true}
				class:default-text={text === 'This is where your cleaned up code will be...'}
			></textarea>
		{:else if activeTab === 'table'}
			<div class="two-column-table">
				<table>
					<thead>
						<tr>
							<th>Original</th>
						</tr>
					</thead>
					<tbody>
						{#each beforeLines as { value, flag }}
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
						{#each afterLines as { value, flag }}
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
		line-height: 0.1; /* Adjust this value as needed */
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
