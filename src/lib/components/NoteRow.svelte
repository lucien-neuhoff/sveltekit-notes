<script lang="ts">
	import type { Note } from '$lib/stores';
	import { createEventDispatcher } from 'svelte';
	import NoteTag from './NoteTag.svelte';
	import { page } from '$app/stores';
	import { goto } from '$app/navigation';
	import { browser } from '$app/environment';

	export let note: Note;
	export let maxNoteContentLengthInDrawer: number = 20;

	const dispatch = createEventDispatcher();

	let hovered: boolean = false;
	let tagJustClicked: boolean = false;

	function formattedNoteContent(note: Note): string {
		return `${note.content.slice(0, maxNoteContentLengthInDrawer)}${
			note.content.length > maxNoteContentLengthInDrawer ? '...' : ''
		}`;
	}
</script>

<li
	on:mouseenter={() => (hovered = true)}
	on:mouseleave={() => (hovered = false)}
	class="rounded-[4px]"
>
	<button
		type="button"
		on:click={() => {
			if (tagJustClicked) return;
			if (browser) goto(`/edit/${note.id}`);
		}}
		class="flex w-full justify-between"
	>
		<span class="flex items-center gap-2 overflow-hidden">
			{formattedNoteContent(note)}
			{#each note.tags as tag, i}
				<NoteTag
					{tag}
					{i}
					on:filter={() => {
						tagJustClicked = true;
						dispatch('filter', tag);
					}}
				/>
			{/each}
		</span>
		<span class="opacity-70">
			<svg
				xmlns="http://www.w3.org/2000/svg"
				fill="none"
				viewBox="0 0 24 24"
				stroke-width="1.5"
				stroke="currentColor"
				class="h-6 w-6"
			>
				<path
					stroke-linecap="round"
					stroke-linejoin="round"
					d="M16.862 4.487l1.687-1.688a1.875 1.875 0 112.652 2.652L10.582 16.07a4.5 4.5 0 01-1.897 1.13L6 18l.8-2.685a4.5 4.5 0 011.13-1.897l8.932-8.931zm0 0L19.5 7.125M18 14v4.75A2.25 2.25 0 0115.75 21H5.25A2.25 2.25 0 013 18.75V8.25A2.25 2.25 0 015.25 6H10"
				/>
			</svg>
		</span>
	</button>
</li>
