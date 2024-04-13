<!-- KeyValue.svelte -->
<script>
    /**
	 * @type {{ key: string, value: string }[]}
	 */
    export let pairs = [];

    export function clear() {
        pairs.length = 0;
    }

    function addPair() {
      pairs = [...pairs, { key: '', value: '' }];
    }
  
    /**
	 * @param {number} index
	 */
    function removePair(index) {
      pairs = pairs.filter((_, i) => i !== index);
    }

  </script>
  
  <div>
    <button class="addargsbutton" on:click={addPair}>Add Args</button>
  </div>
  
  {#each pairs as pair, index}
    <div class="buttonContainer">
      <input type="text" bind:value={pair.key} placeholder="Key">
      <input type="text" bind:value={pair.value} placeholder="Value">
      <button on:click={() => removePair(index)}>Remove</button>
    </div>
  {/each}
  
  <pre>{JSON.stringify(pairs, null, 2)}</pre>

<style>
    button {
		font-size: 14px;
		font-family: 'Roboto', sans-serif;
		margin: 0 10px;
		background-color: #191b22;
		color: rgb(95, 218, 255);
		border-radius: 0px;
	}

    .addargsbutton {
        font-size: 14px;
		font-family: 'Roboto', sans-serif;
		margin: 0 10px;
		background-color: #191b22;
		outline: 1px solid rgb(95, 218, 255);
		color: rgb(95, 218, 255);
		border-radius: 2px;
        margin-bottom: 8px;
        margin-left: 1.85%;
    }

    input {
		border: 2px solid #21242d;
		padding: 1px;
		font-size: 16px;
		background-color: #191b22;
		color: white;
        margin: 2px;
	}

    .buttonContainer {
		display: block; /* Use flexbox */
		justify-content: start; /* Add space between items */
        margin-left: 1.85%;
	}
</style>