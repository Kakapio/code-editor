<!-- TextEditor.svelte -->
<script>
	export let text = '';

	/**
	 * @param {{ target: { value: string; }; }} event
	 */
	export function handleChange(event) {
		text = event.target.value;
	}
</script>

<textarea bind:value={text} readonly={true}></textarea>

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
