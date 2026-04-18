<script lang="ts">
	import { fly, fade } from 'svelte/transition';

	let problem = $state('');
	let assumptions = $state<string[]>(['']);
	let evidence = $state('');
	let options = $state<string[]>(['']);
	let decision = $state('');

	function addAssumption() {
		assumptions = [...assumptions, ''];
	}

	function removeAssumption(index: number) {
		assumptions = assumptions.filter((_, i) => i !== index);
	}

	function addOption() {
		options = [...options, ''];
	}

	function removeOption(index: number) {
		options = options.filter((_, i) => i !== index);
	}
</script>

<div class="mx-auto max-w-[800px] px-4 py-16">
	<h1 class="mb-10 text-3xl font-semibold tracking-tight text-text">Decision Board</h1>

	<div class="flex flex-col gap-6">
		<!-- Problem -->
		<section class="rounded-2xl border border-border bg-surface p-6 shadow-sm">
			<label for="problem" class="mb-2 block text-sm font-medium text-text">Problem</label>
			<textarea
				id="problem"
				bind:value={problem}
				placeholder="What problem are we actually solving?"
				rows={3}
				class="w-full resize-none rounded-xl border border-border bg-bg px-4 py-3 text-sm text-text placeholder-text-secondary outline-none transition-shadow focus:ring-2 focus:ring-primary-soft focus:border-primary"
			></textarea>
		</section>

		<!-- Assumptions -->
		<section class="rounded-2xl border border-border bg-surface p-6 shadow-sm">
			<p class="mb-2 text-sm font-medium text-text">Assumptions</p>
			<div class="flex flex-col gap-2">
				{#each assumptions as assumption, i (i)}
					<div class="flex items-center gap-2" transition:fly={{ y: -8, duration: 200 }}>
						<input
							type="text"
							bind:value={assumptions[i]}
							placeholder="What do we believe is true?"
							class="flex-1 rounded-xl border border-border bg-bg px-4 py-2.5 text-sm text-text placeholder-text-secondary outline-none transition-shadow focus:ring-2 focus:ring-primary-soft focus:border-primary"
						/>
						{#if assumptions.length > 1}
							<button
								onclick={() => removeAssumption(i)}
								class="flex h-8 w-8 shrink-0 cursor-pointer items-center justify-center rounded-lg text-text-secondary transition-colors hover:bg-primary-soft hover:text-primary"
								aria-label="Remove assumption"
							>
								<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
							</button>
						{/if}
					</div>
				{/each}
			</div>
			<button
				onclick={addAssumption}
				class="mt-3 cursor-pointer rounded-lg px-3 py-1.5 text-sm font-medium text-primary transition-colors hover:bg-primary-soft"
			>
				+ Add assumption
			</button>
		</section>

		<!-- Evidence -->
		<section class="rounded-2xl border border-border bg-surface p-6 shadow-sm">
			<label for="evidence" class="mb-2 block text-sm font-medium text-text">Evidence</label>
			<textarea
				id="evidence"
				bind:value={evidence}
				placeholder="What do we know vs think?"
				rows={3}
				class="w-full resize-none rounded-xl border border-border bg-bg px-4 py-3 text-sm text-text placeholder-text-secondary outline-none transition-shadow focus:ring-2 focus:ring-primary-soft focus:border-primary"
			></textarea>
		</section>

		<!-- Options -->
		<section class="rounded-2xl border border-border bg-surface p-6 shadow-sm">
			<p class="mb-2 text-sm font-medium text-text">Options</p>
			<div class="flex flex-col gap-2">
				{#each options as option, i (i)}
					<div class="flex items-center gap-2" transition:fly={{ y: -8, duration: 200 }}>
						<input
							type="text"
							bind:value={options[i]}
							placeholder="What paths can we take?"
							class="flex-1 rounded-xl border border-border bg-bg px-4 py-2.5 text-sm text-text placeholder-text-secondary outline-none transition-shadow focus:ring-2 focus:ring-primary-soft focus:border-primary"
						/>
						{#if options.length > 1}
							<button
								onclick={() => removeOption(i)}
								class="flex h-8 w-8 shrink-0 cursor-pointer items-center justify-center rounded-lg text-text-secondary transition-colors hover:bg-primary-soft hover:text-primary"
								aria-label="Remove option"
							>
								<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
							</button>
						{/if}
					</div>
				{/each}
			</div>
			<button
				onclick={addOption}
				class="mt-3 cursor-pointer rounded-lg px-3 py-1.5 text-sm font-medium text-primary transition-colors hover:bg-primary-soft"
			>
				+ Add option
			</button>
		</section>

		<!-- Decision -->
		<section class="rounded-2xl border border-border bg-surface p-6 shadow-sm">
			<label for="decision" class="mb-2 block text-sm font-medium text-text">Decision</label>
			<textarea
				id="decision"
				bind:value={decision}
				placeholder="What are we choosing and why?"
				rows={3}
				class="w-full resize-none rounded-xl border border-border bg-bg px-4 py-3 text-sm text-text placeholder-text-secondary outline-none transition-shadow focus:ring-2 focus:ring-primary-soft focus:border-primary"
			></textarea>
		</section>
	</div>
</div>
