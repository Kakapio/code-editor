<script>

    export let input = '';
    export let output = '';

    let beforeFrames = [];
    let afterFrames = [];

    export function createDiff() {
        console.warn("okayyy we are making diffs");

    }

</script>

<textarea readonly></textarea>
<textarea readonly></textarea>

<style>
textarea {
		width: 45%;
		height: 500px;
		border: 2px solid #21242d;
		padding: 10px;
		font-size: 16px;
		background-color: #191b22;
		color: white;
        resize: none;
	}
</style>