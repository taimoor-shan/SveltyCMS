<script lang="ts">
	// Stores
	import { entryData, mode, saveFunction, translationProgress, shouldShowNextButton } from '@stores/store';

	// Components
	import Fields from '@components/Fields.svelte';
	import ListNode from './ListNode.svelte';

	//ParaglideJS
	import * as m from '@src/paraglide/messages';

	import { currentChild, type FieldType } from '.';
	import { extractData, getFieldName } from '@utils/utils';

	export let field: FieldType;
	let fieldName = getFieldName(field);

	$translationProgress.show = false;

	export let value = $entryData[fieldName];
	export const WidgetData = async () => _data;

	let MENU_CONTAINER: HTMLUListElement;
	let showFields = false;
	let depth = 0;
	let _data: { [key: string]: any; children: any[] } = $mode == 'create' ? null : value;
	let fieldsData = {};
	let saveMode = $mode;

	// MegaMenu Save Layer Next
	async function saveLayer() {
		if (!_data) {
			_data = { ...(await extractData(fieldsData)), children: [] };
		} else if ($mode == 'edit') {
			let _data = await extractData(fieldsData);
			for (let key in _data) {
				$currentChild[key] = _data[key];
			}
		} else if ($mode == 'create' && $currentChild.children) {
			$currentChild.children.push({ ...(await extractData(fieldsData)), children: [] });
		}
		_data = _data;
		// console.log(_data);
		showFields = false;
		mode.set(saveMode);
		depth = 0;
		shouldShowNextButton.set(false);
		$saveFunction.reset();
	}
</script>

{#if !_data}
	<p class="text-center font-bold text-tertiary-500">
		{m.widget_megamenu_title()}
	</p>
{/if}

<!-- First Menu Entry -->
<!-- TODO Fix ON:keyDOWN -->
{#if !_data || showFields}
	{#key depth}
		{(fieldsData = {}) && ''}
		<Fields fields={field.fields[depth]} root={false} bind:fieldsData customData={$currentChild} />
	{/key}
	{(($saveFunction.fn = saveLayer), '')}
{/if}

<!-- Show children -->
{#if _data}
	<ul bind:this={MENU_CONTAINER} class:hidden={depth != 0} class="children MENU_CONTAINER">
		<div class="w-screen"></div>
		<ListNode {MENU_CONTAINER} self={_data} bind:depth bind:showFields maxDepth={field.fields.length} />
	</ul>
{/if}
