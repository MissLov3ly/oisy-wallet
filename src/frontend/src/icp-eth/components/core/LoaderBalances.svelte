<script lang="ts">
	import { loadBalances, loadErc20Balances } from '$eth/services/balance.services';
	import { address } from '$lib/derived/address.derived';
	import { networkEthereum, networkId } from '$lib/derived/network.derived';
	import { erc20Tokens } from '$eth/derived/erc20.derived';
	import { debounce } from '@dfinity/utils';
	import { ERC20_TWIN_TOKENS } from '$env/tokens.erc20.env';
	import { ckEthereumNativeToken, erc20ToCkErc20Enabled } from '$icp-eth/derived/cketh.derived';

	const load = async () => {
		await Promise.allSettled([
			// We might require Ethereum balance on IC network as well given that one can convert ckETH to ETH.
			loadBalances(),
			...[
				$networkEthereum
					? [
							loadErc20Balances({
								address: $address,
								erc20Tokens: $erc20Tokens,
								networkId: $networkId
							})
						]
					: []
			],
			...[
				$erc20ToCkErc20Enabled
					? [
							// We might require Erc20 balance on IC network as well given that one can convert ckErc20 to Erc20.
							// e.g. USDC <> ckUSDC
							loadErc20Balances({
								address: $address,
								erc20Tokens: ERC20_TWIN_TOKENS,
								networkId: $ckEthereumNativeToken.network.id
							})
						]
					: []
			]
		]);
	};

	const debounceLoad = debounce(load);

	$: $address, $erc20Tokens, $networkId, $erc20ToCkErc20Enabled, debounceLoad();
</script>

<slot />
