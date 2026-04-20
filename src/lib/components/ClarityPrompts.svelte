<script lang="ts">
	import { fly } from 'svelte/transition';

	const prompts = [
		'What assumption, if wrong, breaks this?',
		'What are we avoiding deciding?',
		'What evidence do we actually have?',
		'What would make this irrelevant in 6 months?'
	];

	let currentIndex = $state(0);

	function showNext() {
		currentIndex = (currentIndex + 1) % prompts.length;
	}
</script>

<div class="mx-auto max-w-2xl px-4 py-16">
	<h1 class="mb-10 text-3xl font-semibold tracking-tight text-text">Clarity Prompts</h1>

	<div class="rounded-2xl border border-border bg-surface p-6 shadow-sm">
		{#key currentIndex}
			<div
				class="rounded-xl bg-primary-soft px-6 py-8"
				in:fly={{ y: 12, duration: 250 }}
			>
				<p class="text-lg font-medium leading-relaxed text-text">
					{prompts[currentIndex]}
				</p>
			</div>
		{/key}

		<div class="mt-4 flex items-center justify-between">
			<span class="text-xs text-text-secondary">{currentIndex + 1} / {prompts.length}</span>
			<button
				onclick={showNext}
				class="cursor-pointer rounded-xl bg-primary px-4 py-2 text-sm font-medium text-white transition-colors hover:bg-primary-hover"
			>
				Show another prompt
			</button>
		</div>
	</div>
</div>
