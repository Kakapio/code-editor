<script>
	import { onMount } from 'svelte';

	/**
	 * @type {string[]}
	 */
	let options = [];

	export let selectedOption = '';

	async function fetchData() {
		try {
			const response = await fetch(
				'https://5znoj6mlp7xtbjgyofoqzartjm0xdxkp.lambda-url.us-east-1.on.aws/playground/languages'
			);
			if (!response.ok) {
				throw new Error('Network response was not ok.');
			}
			const responseData = await response.json();
			options = responseData;
		} catch (error) {
			console.error('Error fetching data:', error);
		}
	}

	// Grab our languages when we initialize.
	onMount(fetchData);

	// Function to handle selection change
	// @ts-ignore
	function handleSelection(event) {
		selectedOption = event.target.value;
	}
</script>

<div style="display: flex; align-items: center;">
	<select bind:value={selectedOption} on:change={handleSelection}>
		<option value="" selected disabled hidden>Select language</option>
		<!-- Our first default option is the placeholder -->
		{#each options as option}
			<option value={option}>{option}</option>
		{/each}
	</select>
</div>

<!--TODO: for debug <p style="color: white;">Selected option: {selectedOption}</p> -->

<style>
	/* Style the select element */
	select {
		width: 200px;
		padding: 6px;
		font-size: 16px;
		border: 1px solid #21242d;
		border-radius: 5px;
		background-color: #191b22;
		color: white; /* Set text color */
		appearance: none; /* Remove default arrow */
		margin-left: 1.85%;
	}

	/* Style the options */
	option {
		background-color: #191b22;
		color: white;
	}

	/* Style the selected option */
	option:checked {
		background-color: #007bff;
		color: #fff;
	}

	input {
		margin-left: 10px;
		padding: 6px;
		font-size: 16px;
		border: 1px solid #21242d;
		border-radius: 5px;
		background-color: #191b22;
		color: white;
	}
</style>
