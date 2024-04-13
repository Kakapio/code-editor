<script>
	import TextEditor from '../TextEditor.svelte';
	import TextViewer from '../TextViewer.svelte';
	import LangMenu from '../LangMenu.svelte';
	import KeyValue from '../KeyValue.svelte'
	import toast, { Toaster } from 'svelte-french-toast';

	let inputText = '';
	let outputText = 'This is where your cleaned up code will be...';
	let selectedLang = '';

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

	function clearEditor() {
		editorInstance.clear();
		selectedLang = '';
		keyValueInstance.clear();
		toast.success('Cleared editor!');
	}

	function runCleanup() {
		if (selectedLang === '') {
			toast.error("You must select a language.");
			return;
		}

		let postData = {
			source: inputText,
			language: selectedLang,
			stale_flag: selectedFlag,
			value: true
		};
	}
</script>

<div class="buttonContainer">
	<button on:click={runCleanup}>Run</button>
	<button>Step Once</button>
	<button on:click={clearEditor}>Clear</button>
</div>

<LangMenu
	bind:selectedOption={selectedLang}
	bind:this={menuInstance} />

<Toaster />

<KeyValue bind:this={keyValueInstance} />

<!-- We use 'bind:text' here because data flows from Child -> Parent rather than down from Parent -> Child
     The Editor gives data to our main app page, which then passes it to the Viewer as a "prop". -->

<div class="textContainer">
	<TextEditor bind:text={inputText} bind:this={editorInstance} />
	<TextViewer text={outputText} bind:this={viewerInstance} />
</div>

<style>
	/* Set background color on the root element */
	:root {
		background-color: #16161c; /* Example color */
	}

	.textContainer {
		display: flex; /* Use flexbox */
		justify-content: space-around; /* Add space between items */
	}

	.buttonContainer {
		display: flex; /* Use flexbox */
		justify-content: center; /* Add space between items */
		margin: 15px 0px;
	}

	button {
		font-size: 40px;
		font-family: 'Roboto', sans-serif;
		margin: 0 10px;
		background-color: #191b22;
		outline: 1px solid rgb(95, 218, 255);
		color: rgb(95, 218, 255);
		border-radius: 5px;
	}
</style>
