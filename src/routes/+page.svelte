<script lang="ts">
	import { ListCycle } from '$lib/classes/list_cycle';
	import Checkbox from '$lib/components/Checkbox.svelte';
	import NoNote from '$lib/components/NoNote.svelte';
	import NoteCard from '$lib/components/NoteCard.svelte';
	import NoteRow from '$lib/components/NoteRow.svelte';
	import { getFireNotes } from '$lib/db';
	import { noteStore, filterSettings, user, type Note, type FilterSettings } from '$lib/stores';
	import { InputChip } from '@skeletonlabs/skeleton';

	let getNotes: any;
	let displayMode: ListCycle = new ListCycle(['row', 'card']);

	function addFilterTag(event: any): void {
		let tag: string = event.detail;
		if ($filterSettings.tags.includes(tag)) return;
		$filterSettings.tags.push(tag);
		$filterSettings.refresh++;
	}

	user.subscribe(async (user) => {
		if (!user) return;
		const firestoreNotes: Note[] = await getFireNotes(user.uid);
		const notesToAdd: Note[] = firestoreNotes.filter(
			(fireNote) => !$noteStore.some((note) => note.id === fireNote.id)
		);
		$noteStore.push(...notesToAdd);
	});

	async function filterNotes(settings: FilterSettings): Promise<Note[]> {
		let filteredNotes: Note[] = [];
		let _ = noteStore.subscribe(
			(notes) =>
				(filteredNotes = notes.filter((note) => {
					if (settings.tags.length < 1) {
						return true;
					} else if (settings.matchEvery) {
						return settings.tags.every((tag) => note.tags.includes(tag));
					} else {
						return settings.tags.some((tag) => note.tags.includes(tag));
					}
				}))
		);
		return filteredNotes;
	}

	const _ = filterSettings.subscribe((settings: FilterSettings) => {
		getNotes = filterNotes(settings);
	});
</script>

<div class="flex flex-col gap-4">
	{#if !$user}
		<h1 class="text-center italic opacity-60">
			You are not <a href="/auth"><u>signed in</u></a>, notes will not be shared between devices
		</h1>
	{/if}

	{#if $noteStore.length > 0}
		<div class="flex flex-row items-center gap-4">
			<InputChip bind:value={$filterSettings.tags} name="filter-tags" />
			<Checkbox
				label="Match every"
				on:click={() => ($filterSettings.matchEvery = !$filterSettings.matchEvery)}
			/>
			<button
				type="button"
				class="btn variant-glass"
				on:click={() => {
					displayMode.next();
					displayMode.refresh++;
				}}
			>
				Switch to {displayMode.getNextUppered()}
			</button>
		</div>
		<hr />
	{/if}
	<div class="container mx-auto flex h-full flex-col gap-8">
		{#if $noteStore.length > 0}
			{#key displayMode.changed()}
				{#if displayMode.is('card')}
					<div class="grid grid-cols-2 gap-4 lg:grid-cols-3">
						{#await getNotes then notes}
							{#each notes as note}
								<NoteCard {note} on:filter={addFilterTag} />
							{/each}
						{/await}
					</div>
				{:else if displayMode.is('row')}
					<ul class="list-nav w-full gap-4 space-y-3">
						{#await getNotes then notes}
							{#each notes as note}
								<NoteRow {note} on:filter={addFilterTag} />
							{/each}
						{/await}
					</ul>
				{/if}
			{/key}
		{:else}
			<NoNote />
		{/if}
	</div>
</div>
