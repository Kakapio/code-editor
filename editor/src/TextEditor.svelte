<!-- TextEditor.svelte -->
<script>
	export let text = '';

	// @ts-ignore
	function handleChange(event) {
		text = event.target.value;
	}

	export function clear() {
		text = '';
	}
</script>

<textarea bind:value={text} on:input={handleChange}></textarea>

<style>
	textarea {
		width: 45%;
		height: 500px;
		border: 2px solid #21242d;
		padding: 10px;
		font-size: 16px;
		background-color: #191b22;
		color: white;
	}
</style>
