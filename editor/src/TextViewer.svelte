<!-- TextEditor.svelte -->
<script>
	export let text = '';
	
	/**
	 * @type {{ [x: string]: any; }[]}
	 */
	 export let rewrites = [];

	let originalText = '';
	let activeTab = 'textarea';
	/**
	 * @type {string[]}
	 */
	let beforeLines = [];

	/**
	 * @param {{ target: { value: string; }; }} event
	 */
	export function handleChange(event) {
		text = event.target.value;
	}

	/**
	 * @param {string} text
	 * This is only run when we successfully generate cleaned up code.
	 */
	 export function setOriginalText(text) {
		originalText = text;
		beforeLines = originalText.split("\n");
	}

	export function clear() {
		text = 'This is where your cleaned up code will be...';
	}

	/**
	 * @param {string} tabName
	 */
	function switchTab(tabName) {
		activeTab = tabName;
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
		<table>
			<tbody>
			  {#each beforeLines as line}
				<tr>
				  <td><pre class="code">{line}</pre></td>
				</tr>
			  {/each}
			</tbody>
		  </table>
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
	}

	.code {
    line-height: 0.1; /* Adjust this value as needed */
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

	/* Define custom scrollbar colors for entire site */
	:global(html) {
		scrollbar-color: #21242d #191b22; /* thumb color, track color */
	}
</style>
