<!-- TextEditor.svelte -->
<script>
	export let text = '';

	// @ts-ignore
	export function handleChange(event) {
		text = event.target.value;
	}
</script>

<textarea bind:value={text} readonly={true}></textarea>

<style>
	/* Add your CSS styles here */
	textarea {
		width: 45%;
		height: 500px;
		border: 2px solid #21242D;
		padding: 10px;
		font-size: 16px;
		background-color: #191B22;
		color: white;
	}
</style>
