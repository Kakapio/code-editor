<!-- TextEditor.svelte -->
<script>
	export let text = '';
	let activeTab = 'textarea';

	/**
	 * @param {{ target: { value: string; }; }} event
	 */
	export function handleChange(event) {
		text = event.target.value;
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
        <button on:click={() => switchTab('textarea')} class:selected={activeTab === 'textarea'}>Normal</button>
        <button on:click={() => switchTab('table')} class:selected={activeTab === 'table'}>Diff</button>
    </div>
    <div class="tab-content">
        {#if activeTab === 'textarea'}
            <textarea bind:value={text} readonly={true} class:default-text={text === 'This is where your cleaned up code will be...'}></textarea>
        {:else if activeTab === 'table'}
            <table>
                <!-- Table content goes here -->
                <tr>
                    <th>Column 1</th>
                    <th>Column 2</th>
                    <th>Column 3</th>
                </tr>
                <tr>
                    <td>Data 1</td>
                    <td>Data 2</td>
                    <td>Data 3</td>
                </tr>
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

	button {
		font-size: 14px;
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
		width: 45%;
		margin-top: -21px
    }

    .tab-buttons {
        display: flex;
        justify-content: start;
    }

    .tab-content {
        flex-grow: 1;
    }
</style>
