<script>
  import { onMount } from 'svelte';
  import { MetaMaskSDK } from '@metamask/sdk';
  import { ethers } from 'ethers';

  let username = '';
  let gameResult = null;
  let error = null;
  let myWalletAddress = ''; // Variable to store the user's wallet address
  let opponentAddress = '';
  let gameStart = false;
  let wagerAmount = 0;
  let provider;
  let signer;
  let contract;

  const abi = [
    {
      "inputs": [
        {
          "internalType": "uint256",
          "name": "_id",
          "type": "uint256"
        },
        {
          "internalType": "enum Escrow.ArbitratorDecision",
          "name": "_decision",
          "type": "uint8"
        }
      ],
      "name": "complete",
      "outputs": [],
      "stateMutability": "nonpayable",
      "type": "function"
    },
    {
      "inputs": [
        {
          "internalType": "uint256",
          "name": "_id",
          "type": "uint256"
        }
      ],
      "name": "deposit",
      "outputs": [],
      "stateMutability": "payable",
      "type": "function"
    },
    {
      "inputs": [
        {
          "internalType": "address",
          "name": "_bob",
          "type": "address"
        },
        {
          "internalType": "address",
          "name": "_alice",
          "type": "address"
        },
        {
          "internalType": "uint256",
          "name": "_amount",
          "type": "uint256"
        }
      ],
      "name": "newAgreement",
      "outputs": [
        {
          "internalType": "uint256",
          "name": "",
          "type": "uint256"
        }
      ],
      "stateMutability": "nonpayable",
      "type": "function"
    },
    {
      "inputs": [
        {
          "internalType": "uint256",
          "name": "_id",
          "type": "uint256"
        }
      ],
      "name": "refund",
      "outputs": [],
      "stateMutability": "nonpayable",
      "type": "function"
    },
    {
      "inputs": [
        {
          "internalType": "uint256",
          "name": "",
          "type": "uint256"
        }
      ],
      "name": "agreements",
      "outputs": [
        {
          "internalType": "address",
          "name": "bob",
          "type": "address"
        },
        {
          "internalType": "address",
          "name": "alice",
          "type": "address"
        },
        {
          "internalType": "enum Escrow.ArbitratorDecision",
          "name": "arbitratorDecision",
          "type": "uint8"
        },
        {
          "internalType": "uint256",
          "name": "amount",
          "type": "uint256"
        },
        {
          "internalType": "bool",
          "name": "bobIn",
          "type": "bool"
        },
        {
          "internalType": "bool",
          "name": "aliceIn",
          "type": "bool"
        }
      ],
      "stateMutability": "view",
      "type": "function"
    }
  ];  

  const contractAddress = "0xe3cCEB8F78B38AD2D30ce264C64D4609a2026277"; // Replace with your contract address

  const mmSdk = new MetaMaskSDK({ 
    dappMetadata: {
      name: "Chess Hustler",
      url: window.location.href
    }
    // infuraAPIKey: "YOUR_INFURA_API_KEY" // Optional
  });

  async function connectWallet() {
    if (window.ethereum) {
      // Use the MetaMask SDK to request accounts
      const accounts = await mmSdk.request({ method: "eth_requestAccounts" });
      const userAddress = accounts[0];

      // Set up provider and signer using the connected account
      provider = new ethers.providers.Web3Provider(window.ethereum);
      signer = provider.getSigner();

      // Check if the input address matches the connected account
      if (myWalletAddress.toLowerCase() === userAddress.toLowerCase()) {
        contract = new ethers.Contract(contractAddress, abi, signer);
        console.log(contract);
        console.log("Connected to MetaMask with address:", myWalletAddress);
      } else {
        error = "Entered wallet address does not match the connected account.";
      }
    } else {
      error = "MetaMask is not installed. Please install it.";
    }
  }

  async function sendChallenge() {
    await connectWallet(); // Ensure wallet is connected before proceeding
    if (!myWalletAddress || !opponentAddress || !wagerAmount) {
      error = "Please fill in all fields.";
      return;
    }
    try {
      const tx = await contract.sendChallenge(opponentAddress, wagerAmount); // Replace with your contract method
      await tx.wait();
      console.log("Challenge sent!");
      // Reset fields or perform any other action after sending the challenge
    } catch (err) {
      console.error("Transaction failed:", err);
      error = `Failed to send challenge: ${err.message}`;
    }
  }

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
    <input type="text" bind:value={myWalletAddress} placeholder="Enter Your Wallet Address" />
    <input type="text" bind:value={opponentAddress} placeholder="Enter Opponent's Wallet Address" />
    <label for="wagerInput">Wager Amount: (Pol):</label>
    <input type="number" name="wagerInput" id="wagerInput" bind:value={wagerAmount} placeholder="Enter Wager Amount" />
    <button on:click={sendChallenge}>Send Challenge!</button>
  {:else}
    <button on:click={fetchGameResult}>Refresh Game Results</button>
    <button on:click={() => { gameStart = false; }}>Back</button>
  {/if}

  {#if error}
    <p class="error">{error}</p>
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

  