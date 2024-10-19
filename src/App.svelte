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
  <body>
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
		  <button on:click={() => {
			gameStart = false;
		  }}>Back</button>
		{/if}
	  
		{#if gameResult}
		  <h1>Results</h1>
		  <h2>{gameResult.Black}</h2>
		  <p>vs</p>
		  <h2>{gameResult.White}</h2>
		  <h2>{gameResult.Winner} Wins!</h2>
		{/if}
  </body>
  
  <style>
	*{
		margin: 0;
		padding: 0;
	}
	body{
		padding: 1rem 5rem 8rem;
		border: 3px solid #171513;
		background: rgb(22,21,19);
	background: linear-gradient(0deg, rgba(22,21,19,1) 0%, rgba(45,41,35,1) 100%);
		text-align: center;
		color: #BABABA;
		font-family: "Noto Sans", sans-serif;
	}
	input{
		margin: 5%;
		width: 15rem;
		padding: 3%;
		background: #262422;
		border: none;
		border-bottom: 1px solid #D14D00;
	}
	input:focus{
		outline: none;
	}

	button{
		margin: 5%;
		width: 8rem;
		padding: 3%;
		border: none;
		border-bottom: 1px solid #507C1D;
		color: #cfcfcf;
		background: #262422;
	}

	/* Add some basic styles */
	.error {
	  color: red;
	}
  </style>
  