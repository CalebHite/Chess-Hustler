<script>
	import { onMount } from 'svelte';
  
	let username = ''; // Variable to store the username input
	let gameResult = null; // Variable to store the game result
	let error = null; // Variable to store any errors
	let opponentAddress = '';
	let gameStart = false;
	let wagerAmount = 0;
  
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
		  <label for="wagerInput">Wager Amount: (Pol):</label>
		  <input type="number" name="wagerInput" id="wagerInput" bind:value={wagerAmount} placeholder="Enter Opponent's Wallet Address" />
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
		  <div class="results">
			<h1>Results</h1>
		  <h2 class="black">{gameResult.Black}</h2>
		  <p>vs</p>
		  <h2 class="white">{gameResult.White}</h2>
		  <h2>{gameResult.Winner} Wins!</h2>
		  </div>
		{/if}
  </body>
  
  <style>

  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  body {
    padding: 1rem 5rem 8rem;
    border: 3px solid #171513;
    background: rgb(22, 21, 19);
    background: linear-gradient(0deg, rgba(22, 21, 19, 1) 0%, rgba(45, 41, 35, 1) 100%);
    text-align: center;
    color: #BABABA;
    font-family: "Noto Sans", sans-serif;
  }

  input {
    margin: 5%;
    width: 15rem;
    padding: 3%;
    background: #262422;
    border: none;
    border-bottom: 1px solid #D14D00;
    color: white;
  }

  input:focus {
    outline: none;
    border-top: 1px solid white;
    color: white;
  }

  button {
    margin: 5%;
    width: 8rem;
    padding: 3%;
    border: none;
    border-bottom: 1px solid #507C1D;
    color: #cfcfcf;
    background: #262422;
    cursor: pointer;
  }

  button:hover {
    border-top: 1px solid white;
  }

  .error {
    color: red;
    margin-top: 1rem;
  }

  .results {
    background-color: #5b5751;
    border-radius: 10px;
    padding: 2rem;
    margin-top: 2rem;
    border: 2px solid #D14D00;
    color: #f4f4f4;
  }

  .results h1 {
    font-size: 2rem;
    margin-bottom: 1rem;
    color: #D14D00;
  }

  .results h2 {
    font-size: 1.5rem;
    margin: 0.5rem 0;
    color: #507C1D;
  }

  .results p {
    font-size: 1.2rem;
    margin: 1rem 0;
    color: #BABABA;
  }

  .results .white{
	color: white;
  }

  .results .black{
	color: black;
  }

  .results h2:last-of-type {
    color: #FF4500; /* Highlight the winner */
  }
</style>

  