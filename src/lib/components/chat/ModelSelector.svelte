<script lang="ts">
	import { models, showSettings, settings, user, mobile, config } from '$lib/stores';
	import { thinkingBudget } from '$lib/stores/thinking';
	import { onMount, tick, getContext } from 'svelte';
	import { toast } from 'svelte-sonner';
	import Selector from './ModelSelector/Selector.svelte';
	import Tooltip from '../common/Tooltip.svelte';
	import ChevronDown from '$lib/components/icons/ChevronDown.svelte';

	import { updateUserSettings } from '$lib/apis/users';
	import equal from 'fast-deep-equal';
	const i18n = getContext('i18n');

	// --- REASONING EFFORT STATE WITH VIEWPORT TRACKING ---
	let showReasoningMenu = false;
	let menuCoords = { top: 0, left: 0 }; // Stores live button coordinates

	const budgets = [
		{ value: 0, label: 'Flash' },
		{ value: 1024, label: 'Standard Thinking' },
		{ value: 2048, label: 'Extended Thinking' },
		{ value: 4096, label: 'Deep Reasoning' }
	];

	function toggleReasoningMenu(event: MouseEvent) {
		showReasoningMenu = !showReasoningMenu;
		if (showReasoningMenu && event.currentTarget) {
			// Measures exactly where the button is on your screen right now
			const rect = (event.currentTarget as HTMLElement).getBoundingClientRect();
			menuCoords = {
				top: rect.bottom + 4, // Places it cleanly 4px below the button
				left: rect.left
			};
		}
	}

	function selectBudget(value: number) {
		$thinkingBudget = value;
		showReasoningMenu = false;
	}

	export let selectedModels = [''];
	export let disabled = false;

	export let showSetDefault = true;

	const saveDefaultModel = async () => {
		const hasEmptyModel = selectedModels.filter((it) => it === '');
		if (hasEmptyModel.length) {
			toast.error($i18n.t('Choose a model before saving...'));
			return;
		}
		settings.set({ ...$settings, models: selectedModels });
		await updateUserSettings(localStorage.token, { ui: $settings });

		toast.success($i18n.t('Default model updated'));
	};

	const pinModelHandler = async (modelId) => {
		let pinnedModels = $settings?.pinnedModels ?? [];

		if (pinnedModels.includes(modelId)) {
			pinnedModels = pinnedModels.filter((id) => id !== modelId);
		} else {
			pinnedModels = [...new Set([...pinnedModels, modelId])];
		}

		settings.set({ ...$settings, pinnedModels: pinnedModels });
		await updateUserSettings(localStorage.token, { ui: $settings });
	};

	$: if (selectedModels.length > 0 && $models.length > 0) {
		const _selectedModels = selectedModels.map((model) =>
			$models.map((m) => m.id).includes(model) ? model : ''
		);

		if (!equal(_selectedModels, selectedModels)) {
			selectedModels = _selectedModels;
		}
	}
</script>

<div class="flex flex-col w-full items-start">
	{#each selectedModels as selectedModel, selectedModelIdx}
		<div class="flex w-full max-w-fit">
			<div class="overflow-hidden w-full">
				<div class="max-w-full {($settings?.highContrastMode ?? false) ? 'm-1' : 'mr-1'}">
					<Selector
						id={`${selectedModelIdx}`}
						placeholder={$i18n.t('Select a model')}
						items={$models.map((model) => ({
							value: model.id,
							label: model.name,
							model: model
						}))}
						{pinModelHandler}
						bind:value={selectedModel}
					/>
				</div>
			</div>

	<div class="relative inline-block text-left">
			<button
				type="button"
				on:click|stopPropagation|preventDefault={toggleReasoningMenu}
				class="-mt-[2px] flex items-center w-full text-left px-3 py-1 text-lg text-gray-300 dark:text-neutral-200 bg-transparent hover:bg-gray-100 dark:hover:bg-neutral-800/60 rounded-lg transition h-[32px] whitespace-nowrap min-w-max"
			>
				<span>Thinking Level</span>
				<ChevronDown className="self-center ml-2 size-3" strokeWidth="2.5" />
			</button>

			{#if showReasoningMenu}
				<div class="fixed inset-0 z-[99998]" on:click|stopPropagation|preventDefault={() => showReasoningMenu = false}></div>

				<div 
					class="fixed w-auto min-w-[190px] max-w-xs rounded-xl bg-white dark:bg-[#171717] border border-gray-100 dark:border-neutral-800/70 shadow-2xl z-[99999] overflow-hidden focus:outline-none whitespace-nowrap p-1"
					style="top: {menuCoords.top}px; left: {menuCoords.left}px;"
				>
					<div class="space-y-0.5">
						{#each budgets as item}
							<button
								type="button"
								on:click|stopPropagation|preventDefault={() => selectBudget(item.value)}
								class="w-full text-left px-3 py-2 text-sm transition flex items-center justify-between gap-4 rounded-lg
									{$thinkingBudget === item.value 
										? 'bg-gray-100 dark:bg-[#262626] font-medium text-gray-900 dark:text-white' 
										: 'text-gray-600 dark:text-neutral-400 hover:bg-gray-50 dark:hover:bg-[#262626]/50 hover:text-gray-900 dark:hover:text-neutral-200'}"
							>
								<span>{item.label}</span>
								{#if $thinkingBudget === item.value}
									<svg class="w-4 h-4 text-gray-900 dark:text-white flex-shrink-0" viewBox="0 0 20 20" fill="currentColor">
										<path fill-rule="evenodd" d="M16.704 4.153a.75.75 0 01.143 1.052l-8 10.5a.75.75 0 01-1.127.075l-4.5-4.5a.75.75 0 011.06-1.06l3.894 3.893 7.48-9.817a.75.75 0 011.05-.143z" clip-rule="evenodd" />
									</svg>
								{/if}
							</button>
						{/each}
					</div>
				</div>
			{/if}
		</div>

			{#if $user?.role === 'admin' || ($user?.permissions?.chat?.multiple_models ?? true)}
				{#if selectedModelIdx === 0}
					<div
						class="  self-center mx-1 disabled:text-gray-600 disabled:hover:text-gray-600 -translate-y-[0.5px]"
					>
						<Tooltip content={$i18n.t('Add Model')}>
							<button
								class=" "
								{disabled}
								on:click={() => {
									selectedModels = [...selectedModels, ''];
								}}
								aria-label="Add Model"
							>
								<svg
									xmlns="http://www.w3.org/2000/svg"
									fill="none"
									viewBox="0 0 24 24"
									stroke-width="2"
									stroke="currentColor"
									class="size-3.5"
								>
									<path stroke-linecap="round" stroke-linejoin="round" d="M12 6v12m6-6H6" />
								</svg>
							</button>
						</Tooltip>
					</div>
				{:else}
					<div
						class="  self-center mx-1 disabled:text-gray-600 disabled:hover:text-gray-600 -translate-y-[0.5px]"
					>
						<Tooltip content={$i18n.t('Remove Model')}>
							<button
								{disabled}
								on:click={() => {
									selectedModels.splice(selectedModelIdx, 1);
									selectedModels = selectedModels;
								}}
								aria-label="Remove Model"
							>
								<svg
									xmlns="http://www.w3.org/2000/svg"
									fill="none"
									viewBox="0 0 24 24"
									stroke-width="2"
									stroke="currentColor"
									class="size-3"
								>
									<path stroke-linecap="round" stroke-linejoin="round" d="M19.5 12h-15" />
								</svg>
							</button>
						</Tooltip>
					</div>
				{/if}
			{/if}
		</div>
	{/each}
</div>

{#if showSetDefault}
	<div
		class="relative text-left mt-[1px] ml-1 text-[0.7rem] text-gray-600 dark:text-gray-400 font-primary"
	>
		<button on:click={saveDefaultModel}> {$i18n.t('Set as default')}</button>
	</div>
{/if}
