<script lang="ts">
	import { WizardModal, type WizardStep, type WizardSteps } from '@dfinity/gix-components';
	import { SendStep } from '$lib/enums/steps';
	import HowToConvertEthereumInfo from '$icp/components/convert/HowToConvertEthereumInfo.svelte';
	import type { Network } from '$lib/types/network';
	import ConvertETHToCkETHWizard from '$icp-eth/components/send/ConvertETHToCkETHWizard.svelte';
	import { howToConvertWizardSteps } from '$icp-eth/config/how-to-convert.config';
	import { closeModal } from '$lib/utils/modal.utils';
	import { ICP_NETWORK } from '$env/networks.env';
	import {
		ckEthereumTwinTokenStandard,
		ckEthereumTwinToken,
		ckEthereumNativeTokenId,
		ckEthereumNativeToken
	} from '$icp-eth/derived/cketh.derived';
	import { i18n } from '$lib/stores/i18n.store';
	import {
		toCkErc20HelperContractAddress,
		toCkEthHelperContractAddress
	} from '$icp-eth/utils/cketh.utils';
	import { ckEthMinterInfoStore } from '$icp-eth/stores/cketh.store';

	/**
	 * Props
	 */

	let destination = '';
	$: destination =
		$ckEthereumTwinTokenStandard === 'erc20'
			? toCkErc20HelperContractAddress($ckEthMinterInfoStore?.[$ckEthereumNativeTokenId]) ?? ''
			: toCkEthHelperContractAddress(
					$ckEthMinterInfoStore?.[$ckEthereumNativeTokenId],
					$ckEthereumNativeToken.network.id
				) ?? '';

	let targetNetwork: Network | undefined = ICP_NETWORK;

	let amount: number | undefined = undefined;
	let sendProgressStep: string = SendStep.INITIALIZATION;

	/**
	 * Wizard modal
	 */

	let steps: WizardSteps;
	$: steps = howToConvertWizardSteps({ i18n: $i18n, twinToken: $ckEthereumTwinToken });

	let currentStep: WizardStep | undefined;
	let modal: WizardModal;

	const close = () =>
		closeModal(() => {
			destination = '';
			amount = undefined;
			targetNetwork = undefined;

			sendProgressStep = SendStep.INITIALIZATION;

			currentStep = undefined;
		});
</script>

<WizardModal
	{steps}
	bind:currentStep
	bind:this={modal}
	on:nnsClose={close}
	disablePointerEvents={currentStep?.name === 'Sending'}
>
	<svelte:fragment slot="title">{currentStep?.title ?? ''}</svelte:fragment>

	<ConvertETHToCkETHWizard
		on:icBack={modal.back}
		on:icNext={modal.next}
		on:icClose={close}
		on:icSendBack={() => modal.set(0)}
		bind:destination
		bind:targetNetwork
		bind:amount
		bind:sendProgressStep
		{currentStep}
	>
		<HowToConvertEthereumInfo
			on:icQRCode={modal.next}
			on:icConvert={() => modal.set(2)}
			formCancelAction="close"
		/>
	</ConvertETHToCkETHWizard>
</WizardModal>
