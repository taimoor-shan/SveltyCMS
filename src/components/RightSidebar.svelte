<script lang="ts">
	// Stores
	import { collection, collectionValue, mode, modifyEntry, saveLayerStore, shouldShowNextButton, entryData } from '@stores/store';
	import { handleSidebarToggle } from '@src/stores/sidebarStore';
	import { page } from '$app/stores';

	// Skeleton
	import { Autocomplete, popup } from '@skeletonlabs/skeleton';
	import type { AutocompleteOption, PopupSettings } from '@skeletonlabs/skeleton';

	// Components
	import Toggles from './system/inputs/Toggles.svelte';

	//ParaglideJS
	import * as m from '@src/paraglide/messages';

	import { saveFormData, convertTimestampToDateString } from '@utils/utils';

	let next = () => {};
	saveLayerStore.subscribe((value) => {
		next = value;
		shouldShowNextButton.set(false);
	});

	//const user = 'admin';
	const user = $page.data.user;

	// Map the status to boolean
	//console.log('Status', $entryData?.status);
	let isPublished = $entryData?.status === 'published';
	//console.log('isPublished', isPublished);

	// Function to toggle the status
	function toggleStatus() {
		isPublished = !isPublished;
		$entryData.status = isPublished ? 'published' : 'unpublished';
		$entryData.updatedAt = new Date();
		$entryData.save();
	}

	// Convert timestamp to Date string
	$: dates = {
		created: convertTimestampToDateString($entryData.createdAt),
		updated: convertTimestampToDateString($entryData.updatedAt)
	};

	// Save data
	async function saveData() {
		await saveFormData({ data: $collectionValue });
		mode.set('view');
		handleSidebarToggle();
	}
	//console.log('collection', $collection);

	// TODO: Schedule
	let date = new Date();
	let schedule = '';

	// TODO: user autocomplete
	const Userlist: AutocompleteOption<string>[] = [
		{ label: 'Admin', value: 'admin', keywords: 'plain, basic', meta: { healthy: false } },
		{ label: 'Guest', value: 'guest', keywords: 'dark, white', meta: { healthy: false } },
		{ label: 'User', value: 'user', keywords: 'fruit', meta: { healthy: true } }
	];

	let inputPopupUser: string = '';

	let popupSettingsUser: PopupSettings = {
		event: 'focus-click',
		target: 'popupAutocomplete',
		placement: 'right'
	};

	function onPopupUserSelect(e: CustomEvent<AutocompleteOption<string, unknown>>): void {
		throw new Error('Function not implemented.');
	}
</script>

<!-- Desktop Right Sidebar -->
<!-- Check if user has create or write permission -->
{#if ['edit', 'create'].includes($mode) || $page.data.user.role == 'admin'}
	<div class="flex h-full w-full flex-col justify-between px-1 py-2">
		{#if $shouldShowNextButton && $mode === 'create'}
			<button type="button" on:click={next} class="variant-filled-primary btn w-full gap-2">
				<iconify-icon icon="carbon:next-filled" width="24" class="font-extrabold text-white" />
				{m.button_next()}
			</button>
		{:else}
			<header class="flex flex-col items-center justify-center gap-2">
				<!-- Save button -->
				<button
					type="button"
					on:click={saveData}
					disabled={$collection?.permissions?.[user.role]?.write}
					class="variant-filled-primary btn w-full gap-2"
				>
					<iconify-icon icon="material-symbols:save" width="24" class="font-extrabold text-white" />
					Save
				</button>

				<!-- Publish/Unpublish -->
				<div class="gradient-secondary btn w-full gap-2">
					<Toggles
						label={isPublished ? 'Published' : 'Unpublished'}
						labelColor={isPublished ? 'text-primary-500' : 'text-error-500'}
						icon={isPublished ? 'ic:baseline-check-circle' : 'material-symbols:close'}
						bind:value={isPublished}
						on:toggle={toggleStatus}
					/>
				</div>

				{#if $mode == 'edit'}
					<!--Clone -->
					<button
						type="button"
						on:click={() => $modifyEntry('clone')}
						disabled={$collection?.permissions?.[user.role]?.write && $collection?.permissions?.[user.role]?.create}
						class="gradient-secondary gradient-secondary-hover gradient-secondary-focus btn w-full gap-2 text-white"
					>
						<iconify-icon icon="bi:clipboard-data-fill" width="24" />Clone<span class="text-primary-500">{$collection?.name}</span>
					</button>
				{/if}

				{#if $mode == 'edit'}
					<button
						type="button"
						on:click={() => $modifyEntry('delete')}
						disabled={$collection?.permissions?.[user.role]?.delete}
						class="variant-filled-error btn w-full"
					>
						<iconify-icon icon="icomoon-free:bin" width="24" />Delete
					</button>
				{/if}

				<!-- Promote -->
				<!-- <label class="flex items-center space-x-2">
        <p>Promote</p>
        <input class="checkbox" type="checkbox" checked />
      </label> -->
			</header>

			<!-- Publish Options -->
			<main class="mt-2 flex w-full flex-col items-center justify-center gap-2 text-left dark:text-white">
				<p class="w-full border-b text-center font-bold uppercase text-tertiary-500 dark:text-primary-500">Publish Options:</p>

				<!--Authored by autocomplete -->
				<div class="flexflex-col items-center justify-center">
					<p class="">{m.sidebar_authoredby()}</p>
					<div class="relative z-50">
						<!-- add use:popup directive to the element that triggers the popup -->
						<input
							class="autocomplete variant-filled-surface text-sm"
							type="search"
							name="autocomplete-search"
							bind:value={inputPopupUser}
							placeholder="Search..."
							use:popup={popupSettingsUser}
						/>
						<!-- popup element should have a data-popup attribute that matches the target property in your popup settings -->
						<div data-popup="popupAutocomplete ">
							<!-- ensure Autocomplete component is correctly set up -->
							<Autocomplete bind:input={inputPopupUser} options={Userlist} on:selection={onPopupUserSelect} />
						</div>
					</div>
				</div>

				<!--Authored on -->
				<p class="text-left">{m.sidebar_authoredon()}</p>
				<input type="datetime-local" bind:value={schedule} class="variant-filled-surface text-sm" />
			</main>

			<footer class="mb-1 mt-2">
				{#each Object.entries(dates) as [key, value]}
					<div class="flex items-center justify-center gap-2 text-[12px]">
						<!-- Labels -->
						<div class="capitalize">{key}:</div>
						<!-- Data -->
						<div class="font-bold text-tertiary-500 dark:text-primary-500">{value}</div>
					</div>
				{/each}
			</footer>
		{/if}
	</div>
{/if}
