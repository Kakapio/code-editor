<script>
    import * as Diff from 'diff/dist/diff.js';
  
    export let input = '';
    export let output = '';
  
    let beforeDisplay = [];
    let afterDisplay = [];
  
    export function clear() {
        beforeDisplay = [];
        afterDisplay = [];
    }

    export function createDiff() {
      let diff = Diff.diffLines(input, output);
  
      // Clear the display contents
      beforeDisplay = [];
      afterDisplay = [];
  
      diff.forEach(part => {
        // Determine the color based on whether it's an addition, deletion, or common part
        const color = part.added ? 'green' : part.removed ? 'red' : 'grey';

        // Push the part value wrapped in a span with the determined color to the proper array
        if (part.removed) {
            beforeDisplay.push(`<span style="color: ${color}">${part.value}</span>`);
        }
        else if (part.added) {
            afterDisplay.push(`<span style="color: ${color}">${part.value}</span>`);
        }
        else {
            beforeDisplay.push(`<span style="color: ${color}">${part.value}</span>`);
            afterDisplay.push(`<span style="color: ${color}">${part.value}</span>`);
        }
      });
    }
  
    // Call createDiff() initially and whenever input or output changes
    $: createDiff();
  </script>
  
  <pre>
    {@html beforeDisplay.join('')} <!-- Render the display content -->
  </pre>

  <pre>
    {@html afterDisplay.join('')} <!-- Render the display content -->
  </pre>

<style>
	pre {
		width: 45%;
		height: 100%;
		border: 2px solid #21242d;
		padding: 10px;
		font-size: 16px;
		background-color: #191b22;
		color: white;
		resize: none;
	}
</style>
