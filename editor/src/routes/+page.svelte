<script>
	import TextEditor from '../TextEditor.svelte';
	import TextViewer from '../TextViewer.svelte';

	let inputText = '';

    /**
	 * @type {{ clear: () => void; }}
	 */
    let editorInstance;
    let viewerInstance;

    function clearEditor() {
        editorInstance.clear();
    }
</script>

<div class="buttonContainer">
    <button>Run</button>
    <button>Step Once</button>
    <button on:click={clearEditor}>Clear</button>
</div>


<!-- We use 'bind' here because data flows from Child -> Parent rather than down from Parent -> Child
     The Editor gives data to our main app page, which then passes it to the Viewer as a "prop". -->
<div class="textContainer">
	<TextEditor bind:text={inputText} bind:this={editorInstance}/>
	<TextViewer text={inputText} bind:this={viewerInstance}/>
</div>

<style>
	/* Set background color on the root element */
	:root {
		background-color: #16161C; /* Example color */
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
        background-color: #191B22; /* Add red background color */
        outline: 2px solid rgb(0, 195, 255); /* Add green outline */
        color: rgb(0, 195, 255);
        border-radius: 5px;
    }
</style>
