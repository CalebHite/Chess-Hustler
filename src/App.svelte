<script>
	import { onMount } from 'svelte';
  
	let username = ''; // Variable to store the username input
	let gameResult = null; // Variable to store the game result
	let error = null; // Variable to store any errors
	let opponentAddress = '';
	let gameStart = false;
  
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
	{#if !gameStart}
	  <h1>Chess Hustler</h1>
	  <input type="text" bind:value={username} placeholder="Enter username" />
	  <input type="text" bind:value={opponentAddress} placeholder="Enter Opponent's Wallet Address" />
	  <button on:click={() => {
		gameStart = true;
	  }}>Send Challenge!</button>
	{:else}
	  <button on:click={fetchGameResult}>Refresh Game Results</button>
	{/if}
  
	{#if error}
	  <p class="error">{error}</p>
	{/if}
  
	{#if gameResult}
	  <h1>Results</h1>
	  <h2>{gameResult.Black}</h2>
	  <p>vs</p>
	  <h2>{gameResult.White}</h2>
	  <h2>{gameResult.Winner} Wins!</h2>
	{/if}
  </div>
  
  <style>
	/* Add some basic styles */
	.error {
	  color: red;
	}
  </style>
  