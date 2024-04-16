<script>
	import TextEditor from '../TextEditor.svelte';
	import TextViewer from '../TextViewer.svelte';
	import LangMenu from '../LangMenu.svelte';
	import KeyValue from '../KeyValue.svelte';
	import DiffView from '../DiffView.svelte';
	import toast, { Toaster } from 'svelte-french-toast';

	let inputText = '';
	let outputText = '';
	let selectedLang = '';
	/**
	 * @type {{ original: string, range: string }[]}
	 */
	let rewrites = [];
	/**
	 * @type {{ key: string, value: string }[]}
	 */
	let keyValues = [];

	/**
	 * @type {TextEditor}
	 */
	let editorInstance;
	/**
	 * @type {TextViewer}
	 */
	let viewerInstance;
	/**
	 * @type {LangMenu}
	 */
	let menuInstance;
	/**
	 * @type {KeyValue}
	 */
	let keyValueInstance;

	/**
	 * @type {DiffView}
	 */
	let diffViewInstance;

	function clearEditor() {
		editorInstance.clear();
		viewerInstance.clear();
		diffViewInstance.clear();

		outputText = ''; // Data that flows from page -> viewer should be cleared by the page.
		rewrites.length = 0;

		selectedLang = '';
		keyValueInstance.clear();
		toast.success('Cleared editor!');
	}

	function runCleanup() {
		if (selectedLang === '') {
			toast.error('You must select a language.');
			return;
		}
		if (inputText === '') {
			toast.error('You most provide some code.');
			return;
		}

		// Assembling our data...
		let postData = {
			source: inputText,
			language: selectedLang
		};

		keyValues.forEach((item) => {
			postData[item.key] = item.value;
		});
		let jsonCompatibleData = JSON.stringify(postData);
		console.log('payload: ' + jsonCompatibleData);

		fetch('https://5znoj6mlp7xtbjgyofoqzartjm0xdxkp.lambda-url.us-east-1.on.aws/playground/play', {
			method: 'POST',
			body: jsonCompatibleData,
			headers: {
				'Content-type': 'application/json'
			}
		})
			.then((response) => response.json())
			.then((data) => {
				console.log('received: ' + JSON.stringify(data));
				outputText = data['output'];
				rewrites = data['rewrites'];
			})
			.catch((error) => {
				if (error.message.includes('Unexpected')) {
					toast.error(
						'It looks like you forgot to specify the correct arguments. Add the flag and value on the left.'
					);
				} else {
					toast.error('Encountered an error: ' + error.message);
				}
			})
			.then((_) => {
				viewerInstance.createDiff();
				diffViewInstance.createDiff();
			}); // This statement is necessary to make sure the rewrites are in place before making diffs.
	}
</script>

<div class="buttonContainer">
	<button on:click={runCleanup}>Run</button>
	<button on:click={clearEditor}>Clear</button>
</div>

<LangMenu bind:selectedOption={selectedLang} bind:this={menuInstance} />

<Toaster />

<KeyValue bind:pairs={keyValues} bind:this={keyValueInstance} />

<!-- We use 'bind:text' here because data flows from Child -> Parent rather than down from Parent -> Child
     The Editor gives data to our main app page, which then passes it to the Viewer as a "prop". -->

<div class="textContainer">
	<TextEditor bind:text={inputText} bind:this={editorInstance} />
	<TextViewer originalText={inputText} text={outputText} {rewrites} bind:this={viewerInstance} />
</div>

<div class="diffContainer">
	<DiffView input={inputText} output={outputText} bind:this={diffViewInstance} />
</div>

<style>
	/* Set background color on the root element */
	:root {
		background-color: #16161c;
	}

	.textContainer {
		display: flex;
		justify-content: space-evenly;
		margin-top: 10px;
	}

	.diffContainer {
		display: flex;
		justify-content: center;
		margin-top: 10px;
	}

	.buttonContainer {
		display: flex;
		justify-content: center;
		margin: 15px 0px;
	}

	button {
		font-size: 32px;
		font-family: 'Roboto', sans-serif;
		margin: 0 10px;
		background-color: #191b22;
		outline: 1px solid rgb(95, 218, 255);
		color: rgb(95, 218, 255);
		border-radius: 5px;
	}
</style>
