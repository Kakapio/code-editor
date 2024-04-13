<script>
	import { onMount } from 'svelte';

	let options = [];
	let selectedOption = '';

	async function fetchData() {
		try {
			const response = await fetch(
				'https://5znoj6mlp7xtbjgyofoqzartjm0xdxkp.lambda-url.us-east-1.on.aws/playground/languages'
			);
			if (!response.ok) {
				throw new Error('Network response was not ok');
			}
			const responseData = await response.json();
			options = responseData;
			console.log('Raw response body: ', options);
		} catch (error) {
			console.error('Error fetching data:', error);
		}
	}

	// Grab our languages when we initialize.
	onMount(fetchData);

	// Function to handle selection change
	function handleSelection(event) {
		selectedOption = event.target.value;
	}
</script>

<select bind:value={selectedOption} on:change={handleSelection}>
	<option value="">Select an option</option>
	{#each options as option}
		<option value={option}>{option}</option>
	{/each}
</select>

<p color style="color: white;">Selected option: {selectedOption}</p>
