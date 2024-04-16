<script>

    export let input = '';
    export let output = '';

    let beforeFrames = [];
    let afterFrames = [];

    export function createDiff() {
        console.warn("okayyy we are making diffs");
        
    }

</script>

<div class="two-column-table">
    <table>
        <thead>
            <tr>
                <th>Original</th>
            </tr>
        </thead>
        <tbody>
            {#each beforeFrames as { value, flag }}
                <tr>
                    <td><pre class:codered={flag}>{value}</pre></td>
                </tr>
            {/each}
        </tbody>
    </table>

    <table>
        <thead>
            <tr>
                <th>Replacement</th>
            </tr>
        </thead>
        <tbody>
            {#each afterFrames as { value, flag }}
                <tr>
                    <td><pre class:codegreen={flag}>{value}</pre></td>
                </tr>
            {/each}
        </tbody>
    </table>
</div>

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

    td {
		color: white;
		font-family: 'Roboto', sans-serif;
	}

	th {
		color: rgb(95, 218, 255);
		font-family: 'Roboto', sans-serif;
	}

	tr {
		background-color: #191b22;
		line-height: 0.7; /* Adjust this value as needed */
	}

	table {
		width: 100%;
		height: 523px;
		border-spacing: 0; /* Remove gaps between table rows */
		border: 2px solid #21242d;
	}

    .codered {
		line-height: 0.1; /* Adjust this value as needed */
		background-color: #642121;
	}

	.codegreen {
		line-height: 0.1; /* Adjust this value as needed */
		background-color: #246421;
	}

    .two-column-table {
		display: flex;
	}

</style>