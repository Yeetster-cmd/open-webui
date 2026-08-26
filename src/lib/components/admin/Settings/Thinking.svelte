<script lang="ts">
	import { toast } from 'svelte-sonner';
	import { createEventDispatcher, getContext } from 'svelte';
	const dispatch = createEventDispatcher();

	import { updateUserSettings } from '$lib/apis/users';
	import { settings } from '$lib/stores';

	import Textarea from '$lib/components/common/Textarea.svelte';
	import AdminSettingField from './AdminSettingField.svelte';
	import AdminSettingRow from './AdminSettingRow.svelte';
	import AdminSettingSection from './AdminSettingSection.svelte';

	const i18n = getContext('i18n');

	export let saveHandler: () => void;

	const DEFAULT_THINKING_CONFIG = {
		flash: { value: 0, systemPrompt: 'reasoning_effort=None (0%). This is the lowest effort you can utilize. Your reasoning modes are: None, Standard, High, Max.\nAnswer the user in a quick manner without asking for clarifications or overthinking.' },
		standard: { value: 512, systemPrompt: 'reasoning_effort=Standard (33%). This is a balanaced effort of your abilities. Your reasoning modes are: None, Standard, High, Max.\nAnswer the user\'s inquiry normally.' },
		extended: { value: 2048, systemPrompt: 'reasoning_effort=High (66%). This is a higher effort setting of your abilities. Your reasoning modes are: None, Standard, High, Max.\nGo over everything you know about the topic being talked about and answer the user\'s inquiry. Ask for details if you need them.' },
		deep: { value: -1, systemPrompt: 'reasoning_effort=Max (100%). This is the maximum effort setting of your abilities. Your reasoning modes are: None, Standard, High, Max.\nYou are an expert at the given topic being discussed. Go into as much detail as possible, utilize your thinking to double and triple check facts, replies and story information. You should not leave any details out. Create a thorough plan of action to the user\'s inquiry.' }
	};

	let configValues = {
		flash: { ...DEFAULT_THINKING_CONFIG.flash, ...($settings?.thinkingBudgets?.flash ?? {}) },
		standard: { ...DEFAULT_THINKING_CONFIG.standard, ...($settings?.thinkingBudgets?.standard ?? {}) },
		extended: { ...DEFAULT_THINKING_CONFIG.extended, ...($settings?.thinkingBudgets?.extended ?? {}) },
		deep: { ...DEFAULT_THINKING_CONFIG.deep, ...($settings?.thinkingBudgets?.deep ?? {}) },
	};

	const levels = [
		{ key: 'flash', label: 'Flash', description: 'Quickest reply' },
		{ key: 'standard', label: 'Standard', description: 'Best for most questions' },
		{ key: 'extended', label: 'Extended', description: 'Complex problem solving' },
		{ key: 'deep', label: 'Deep Reasoning', description: 'Get detailed reports' }
	];

	function reset() {
		configValues = JSON.parse(JSON.stringify(DEFAULT_THINKING_CONFIG));
	}

	async function save() {
		const updatedSettings = { ...$settings, thinkingBudgets: configValues };
		settings.set(updatedSettings);
		await updateUserSettings(localStorage.token, { ui: updatedSettings });
		toast.success($i18n.t('Thinking settings saved'));
		saveHandler();
	}
</script>

<div class="flex flex-col gap-5">
	<AdminSettingSection title="Tokens per Level" first>
		{#each levels as level}
			<AdminSettingRow label={level.label} description={level.description}>
				<input
					type="number"
					min="0"
					step="256"
					bind:value={configValues[level.key].value}
					class="w-28 h-7 rounded-lg border border-gray-100/50 bg-gray-50/40 px-2 text-xs text-right text-gray-700 outline-hidden transition-colors dark:border-white/[0.04] dark:bg-white/[0.03] dark:text-gray-300"
				/>
			</AdminSettingRow>
		{/each}
	</AdminSettingSection>

	<AdminSettingSection title="System Prompts">
		{#each levels as level}
			<AdminSettingField label={level.label}>
				<Textarea
					bind:value={configValues[level.key].systemPrompt}
					rows="3"
					placeholder="No prompt"
					className="w-full resize-y rounded-lg border border-gray-100/50 bg-gray-50/40 px-2 py-1.5 text-xs text-gray-700 outline-hidden transition-colors dark:border-white/[0.04] dark:bg-white/[0.03] dark:text-gray-300"
				/>
			</AdminSettingField>
		{/each}
		<p class="text-[0.6875rem] text-gray-400 dark:text-gray-600">
			Prompts are sent as an extra system message when the matching thinking level is active. Leave empty to disable.
		</p>
	</AdminSettingSection>

	<div class="flex gap-2 justify-end">
		<button
			type="button"
			on:click={reset}
			class="px-3 py-1.5 text-sm rounded-lg bg-gray-100 dark:bg-gray-800 text-gray-600 dark:text-gray-300 hover:bg-gray-200 dark:hover:bg-gray-700 transition-colors"
		>
			{$i18n.t('Reset')}
		</button>
		<button
			type="button"
			on:click={save}
			class="px-3 py-1.5 text-sm rounded-lg bg-gray-900 dark:bg-gray-100 text-white dark:text-gray-900 hover:bg-gray-700 dark:hover:bg-white transition-colors"
		>
			{$i18n.t('Save')}
		</button>
	</div>
</div>
