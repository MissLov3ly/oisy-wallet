<script lang="ts">
	import TokenMenu from '$lib/components/tokens/TokenMenu.svelte';
	import { i18n } from '$lib/stores/i18n.store';
	import ExternalLink from '$lib/components/ui/ExternalLink.svelte';
	import { nonNullish } from '@dfinity/utils';
	import { token } from '$lib/derived/token.derived';
	import { fade } from 'svelte/transition';
	import type { IcCkToken } from '$icp/types/ic';

	let explorerUrl: string | undefined;
	$: explorerUrl = ($token as IcCkToken).explorerUrl;

	let transactionsExplorerUrl: string | undefined;
	$: transactionsExplorerUrl = nonNullish(explorerUrl) ? `${explorerUrl}/transactions` : undefined;
</script>

<TokenMenu>
	{#if nonNullish(transactionsExplorerUrl)}
		<div in:fade>
			<ExternalLink
				href={transactionsExplorerUrl}
				ariaLabel={$i18n.tokens.alt.open_dashboard}
				iconVisible={false}
			>
				{$i18n.navigation.text.view_on_explorer}
			</ExternalLink>
		</div>
	{/if}
</TokenMenu>
