<script lang="ts">
	import { onMount } from 'svelte';
	import { Contract, ethers, Signer, type ContractInterface } from 'ethers';
    
	import abi from '$lib/abi/LW3Punks.json';
	import '../app.css';
	import punk from '$lib/assets/1.png';	

	let tokenIdsMinted: String;
	let account: string;
	let loading: boolean;

	onMount(async () => {
		try {
			account = await checkIfWalletIsConnected();
			let networkChainId = await getNetwork();
			let rinkebyChainId = '0x4';

			if (networkChainId != rinkebyChainId) {
				await switchNetwork(rinkebyChainId);
			}

			if (networkChainId == rinkebyChainId) {
				tokenIdsMinted = await getTokenIdsMinted();
			}
		} catch (error) {
			console.log(error);
		}
	});

	export function getContractAddress() {
		return '0xDF4480a8df3e198407C93A8E7d0789f7ab7B3B83';
	}

	export async function checkIfWalletIsConnected(): Promise<string> {
		let account: string;

		try {
			if (!window.ethereum) {
				console.log('Make sure you have metamask!');
				return '';
			} else {
				console.log('We have the ethereum object', window.ethereum);
			}

			const accounts: string[] = await window.ethereum.request({ method: 'eth_accounts' });

			if (accounts.length !== 0) {
				account = accounts[0];
				console.log('Found an authorized account:', account);
				return account;
			} else {
				console.log('No authorized account found');
			}
		} catch (error) {
			throw error;
		}
	}

	export async function connectWallet(): Promise<void> {
		try {
			if (!window.ethereum) {
				alert('Make sure you have metamask!');
				return;
			}

			const accounts: string[] = await window.ethereum.request({ method: 'eth_requestAccounts' });

			if (accounts.length !== 0) {
				account = accounts[0];
				console.log('Found an authorized account:', account);
			} else {
				console.log('No authorized account found');
			}
		} catch (error) {
			throw error;
		}
	}

	export async function getNetwork(): Promise<string> {
		let network: string;

		try {
			if (!window.ethereum) {
				alert('Make sure you have metamask!');
				return;
			}

			const chainId = await window.ethereum.request({ method: 'eth_chainId' });
			network = chainId;
		} catch (error) {
			throw error;
		}

		return network;
	}

	export async function switchNetwork(chainId: string): Promise<void> {
		try {
			if (window.ethereum) {
				await window.ethereum.request({
					method: 'wallet_switchEthereumChain',
					params: [{ chainId: chainId }]
				});
			} else {
				alert(
					'MetaMask is not installed. Please install it to use this app: https://metamask.io/download.html'
				);
			}
		} catch (error) {
			throw error;
		}
	}

	function getContract(): Contract {
		let LW3PunksContract: Contract;

		try {
			const signer: Signer = getSigner();
			let contractABI: ContractInterface = abi.abi;
			let contractAddress: string = getContractAddress();

			if (signer) {
				LW3PunksContract = new ethers.Contract(contractAddress, contractABI, signer);
				console.log('LW3PunksContract', LW3PunksContract.address);
			}
		} catch (error) {
			console.log('getContract', error);
		}

		return LW3PunksContract;
	}

	function getSigner(): Signer {
		let signer: Signer;

		try {
			if (window.ethereum) {
				const provider: ethers.providers.Web3Provider = new ethers.providers.Web3Provider(
					window.ethereum
				);
				signer = provider.getSigner();
			}
		} catch (error) {
			console.log('getContract', error);
		}

		return signer;
	}

	export async function getTokenIdsMinted(): Promise<string> {
		let tokenIdsMinted: string;
		try {
			const nftContract = getContract();
			const _tokenIds = await nftContract.tokenIds();
			tokenIdsMinted = _tokenIds.toString();
		} catch (error) {
			throw error;
		}
		return tokenIdsMinted;
	}

	export async function publicMint() {
		try {
			const signer = await getSigner();
			const nftContract = getContract();
			const txn = await nftContract.mint({
				value: ethers.utils.parseEther('0.01')
			});
			loading = true;
			await txn.wait();
			tokenIdsMinted = await getTokenIdsMinted();
			loading = false;
			alert('You successfully minted a LW3Punk!');
		} catch (error) {
			throw error;
		}
	}
</script>

<div>
	<div class="main">
		<div>
			<h1 class="title">Welcome to LW3Punks!</h1>
			<div class="description">Its an NFT collection for LearnWeb3 students.</div>
			<div class="description">
				{tokenIdsMinted}/10 have been minted
			</div>
			{#if !account}
				<button on:click={connectWallet} class="button"> Connect your wallet </button>
			{:else if loading}
				<button class="button">Loading...</button>
			{:else}
				<button class="button" on:click={publicMint}> Public Mint 🚀 </button>
			{/if}
		</div>
		<div>
			<img class="image" src={punk} alt="" />
		</div>
	</div>
	<footer class="footer">Made with &#10084; by LW3Punks</footer>
</div>
