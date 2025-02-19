<script lang="ts">
	import { i18n } from '$lib/stores/i18n.store';
	import { isNullishOrEmpty } from '$lib/utils/input.utils';
	import { toastsError } from '$lib/stores/toasts.store';
	import { setCustomToken } from '$lib/api/backend.api';
	import { token } from '$lib/derived/token.derived';
	import HideTokenModal from '$lib/components/tokens/HideTokenModal.svelte';
	import type { Identity } from '@dfinity/agent';
	import type { LedgerCanisterIdText } from '$icp/types/canister';
	import { assertNonNullish, toNullable } from '@dfinity/utils';
	import { Principal } from '@dfinity/principal';
	import { ICP_NETWORK_ID } from '$env/networks.env';
	import { onMount } from 'svelte';
	import type { IcrcCustomToken } from '$icp/types/icrc-custom-token';
	import { loadUserTokens } from '$icp/services/icrc.services';

	let selectedToken: IcrcCustomToken | undefined;

	// We must clone the reference to avoid the UI to rerender once we remove the token from the store.
	onMount(() => (selectedToken = $token as IcrcCustomToken));

	let ledgerCanisterId: LedgerCanisterIdText | undefined;
	$: ledgerCanisterId = selectedToken?.ledgerCanisterId;

	let indexCanisterId: LedgerCanisterIdText | undefined;
	$: indexCanisterId = selectedToken?.indexCanisterId;

	let version: bigint | undefined;
	$: version = selectedToken?.version;

	const assertHide = (): { valid: boolean } => {
		if (isNullishOrEmpty(ledgerCanisterId)) {
			toastsError({
				msg: { text: $i18n.tokens.error.invalid_ledger }
			});
			return { valid: false };
		}

		if (isNullishOrEmpty(indexCanisterId)) {
			toastsError({
				msg: { text: $i18n.tokens.error.invalid_index }
			});
			return { valid: false };
		}

		return { valid: true };
	};

	const hideToken = async (params: { identity: Identity }) => {
		assertNonNullish(ledgerCanisterId);
		assertNonNullish(indexCanisterId);

		await setCustomToken({
			...params,
			token: {
				enabled: false,
				version: toNullable(version),
				token: {
					Icrc: {
						ledger_id: Principal.fromText(ledgerCanisterId),
						index_id: toNullable(Principal.fromText(indexCanisterId))
					}
				}
			}
		});
	};

	const updateUi = (params: { identity: Identity }): Promise<void> => loadUserTokens(params);
</script>

<HideTokenModal backToNetworkId={ICP_NETWORK_ID} {assertHide} {hideToken} {updateUi} />
