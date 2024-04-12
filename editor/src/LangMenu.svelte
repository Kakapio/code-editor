<script>
    let responseData;
  
    async function fetchData() {
      try {
        const response = await fetch('https://5znoj6mlp7xtbjgyofoqzartjm0xdxkp.lambda-url.us-east-1.on.aws/playground/languages');
        if (!response.ok) {
          throw new Error('Network response was not ok');
        }
        responseData = await response.json();
      } 
      catch (error) {
        console.error('Error fetching data:', error);
      }

      console.log(response);
    }
  </script>
  
  <button on:click={fetchData}>Fetch Data</button>