<script>
	import { onMount } from 'svelte';
  
	let username = ''; // Variable to store the username input
	let gameResult = null; // Variable to store the game result
	let error = null; // Variable to store any errors
  
	// Function to fetch game results
	async function fetchGameResult() {
	  if (!username) {
		error = "Please enter a username.";
		return;
	  }
  
	  try {
		const response = await fetch(`http://localhost:5000/get_game?username=${username}`);
		if (!response.ok) {
		  throw new Error(`HTTP error! status: ${response.status}`);
		}
		gameResult = await response.json();
		error = null; // Clear any previous errors
	  } catch (err) {
		error = `Failed to fetch data: ${err.message}`;
		gameResult = null; // Clear previous game results
	  }
	}
</script>
  
<div>
	<h1>Lichess Game Result</h1>
	<input type="text" bind:value={username} placeholder="Enter username" />
	<button on:click={fetchGameResult}>Get Latest Game</button>
  
	{#if error}
	  <p class="error">{error}</p>
	{/if}
  
	{#if gameResult}
	  <h2>Game Result</h2>
	  <pre>{JSON.stringify(gameResult, null, 2)}</pre>
	{/if}
</div>
  
<style>
	/* Add some basic styles */
	.error {
	  color: red;
	}
</style>
