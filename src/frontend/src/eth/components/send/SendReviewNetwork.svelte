<script lang="ts">
	import type { Network } from '$lib/types/network';
	import { nonNullish } from '@dfinity/utils';
	import Value from '$lib/components/ui/Value.svelte';
	import { isNetworkICP } from '$lib/utils/network.utils';
	import type { EthereumNetwork } from '$eth/types/network';
	import eth from '$icp-eth/assets/eth.svg';
	import Logo from '$lib/components/ui/Logo.svelte';
	import icpDark from '$eth/assets/icp_dark.svg';
	import type { Token } from '$lib/types/token';
	import { i18n } from '$lib/stores/i18n.store';
	import { replacePlaceholders } from '$lib/utils/i18n.utils';
	import { ICP_NETWORK } from '$env/networks.env';
	import type { Erc20Token } from '$eth/types/erc20';
	import { ERC20_CONTRACT_ICP, ERC20_CONTRACT_ICP_GOERLI } from '$env/tokens.erc20.env';

	export let sourceNetwork: EthereumNetwork;
	export let targetNetwork: Network | undefined = undefined;
	export let token: Token;

	let nativeIcp: boolean;
	$: nativeIcp =
		nonNullish(targetNetwork) &&
		isNetworkICP(targetNetwork) &&
		[ERC20_CONTRACT_ICP.address, ERC20_CONTRACT_ICP_GOERLI.address].includes(
			(token as Erc20Token)?.address
		);
</script>

<Value ref="source-network" element="div">
	<svelte:fragment slot="label"
		>{#if nonNullish(targetNetwork)}{$i18n.send.text.source_network}{:else}{$i18n.send.text
				.network}{/if}</svelte:fragment
	>
	<span class="flex gap-1">
		{sourceNetwork.name}
		<Logo src={sourceNetwork.icon ?? eth} size="20px" alt={`${sourceNetwork.name} logo`} />
	</span>
</Value>

{#if nonNullish(targetNetwork)}
	<Value ref="target-network" element="div">
		<svelte:fragment slot="label">{$i18n.send.text.destination_network}</svelte:fragment>
		<span class="flex gap-1">
			{#if nativeIcp}
				{$i18n.send.text.convert_to_native_icp}
				<Logo
					src={icpDark}
					size="20px"
					alt={replacePlaceholders($i18n.core.alt.logo, {
						$name: ICP_NETWORK.name
					})}
				/>
			{:else}
				{targetNetwork.name}
				<Logo
					src={targetNetwork.icon ?? eth}
					size="20px"
					alt={replacePlaceholders($i18n.core.alt.logo, {
						$name: targetNetwork.name
					})}
				/>
			{/if}
		</span>
	</Value>
{/if}
