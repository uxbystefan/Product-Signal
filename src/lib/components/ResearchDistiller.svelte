<script lang="ts">
	let rawNotes = $state('');
	let patterns = $state('');
	let insights = $state(['', '', '']);
	let confidence = $state<'high' | 'medium' | 'low' | ''>('');

	function addInsight() {
		if (insights.length < 5) {
			insights = [...insights, ''];
		}
	}

	function removeInsight(index: number) {
		if (insights.length > 3) {
			insights = insights.filter((_, i) => i !== index);
		}
	}

	let filledInsights = $derived(insights.filter((s) => s.trim()).length);
</script>

<div class="mx-auto max-w-2xl px-4 py-16">
	<h1 class="mb-10 text-3xl font-semibold tracking-tight text-text">Research Distiller</h1>

	<div class="flex flex-col gap-6">
		<!-- Raw Notes -->
		<section class="rounded-2xl border border-border bg-surface p-6 shadow-sm">
			<label for="raw-notes" class="mb-2 block text-sm font-medium text-text">Raw Notes</label>
			<textarea
				id="raw-notes"
				bind:value={rawNotes}
				placeholder="Paste notes, interviews, feedback..."
				rows={5}
				class="w-full resize-none rounded-xl border border-border bg-bg px-4 py-3 text-sm text-text placeholder-text-secondary outline-none transition-shadow focus:ring-2 focus:ring-primary-soft focus:border-primary"
			></textarea>
		</section>

		<!-- Patterns -->
		<section class="rounded-2xl border border-border bg-surface p-6 shadow-sm">
			<label for="patterns" class="mb-2 block text-sm font-medium text-text">Patterns</label>
			<textarea
				id="patterns"
				bind:value={patterns}
				placeholder="What patterns are repeating?"
				rows={3}
				class="w-full resize-none rounded-xl border border-border bg-bg px-4 py-3 text-sm text-text placeholder-text-secondary outline-none transition-shadow focus:ring-2 focus:ring-primary-soft focus:border-primary"
			></textarea>
		</section>

		<!-- Insights -->
		<section class="rounded-2xl border border-border bg-surface p-6 shadow-sm">
			<p class="mb-1 text-sm font-medium text-text">Insights</p>
			<p class="mb-3 text-xs text-text-secondary">Limit to 3–5 insights. If it's not in the top 5, it's not important.</p>
			<div class="flex flex-col gap-2">
				{#each insights as _, i (i)}
					<div class="flex items-center gap-2">
						<input
							type="text"
							bind:value={insights[i]}
							placeholder="Key insight"
							class="flex-1 rounded-xl border border-border bg-bg px-4 py-2.5 text-sm text-text placeholder-text-secondary outline-none transition-shadow focus:ring-2 focus:ring-primary-soft focus:border-primary"
						/>
						{#if insights.length > 3}
							<button
								onclick={() => removeInsight(i)}
								class="flex h-8 w-8 shrink-0 cursor-pointer items-center justify-center rounded-lg text-text-secondary transition-colors hover:bg-primary-soft hover:text-primary"
								aria-label="Remove insight"
							>
								<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
							</button>
						{/if}
					</div>
				{/each}
			</div>
			{#if filledInsights < 3}
				<p class="mt-2 text-xs text-amber-600">At least 3 insights required</p>
			{/if}
			<button
				onclick={addInsight}
				disabled={insights.length >= 5}
				class="mt-3 cursor-pointer rounded-lg px-3 py-1.5 text-sm font-medium text-primary transition-colors hover:bg-primary-soft disabled:cursor-not-allowed disabled:opacity-40"
			>
				+ Add insight
			</button>
		</section>

		<!-- Confidence -->
		<section class="rounded-2xl border border-border bg-surface p-6 shadow-sm">
			<p class="mb-3 text-sm font-medium text-text">Confidence</p>
			<div class="flex gap-2">
				{#each ['high', 'medium', 'low'] as level (level)}
					<button
						onclick={() => (confidence = level as 'high' | 'medium' | 'low')}
						class="cursor-pointer rounded-xl border px-4 py-2 text-sm font-medium capitalize transition-colors
							{confidence === level
								? 'border-primary bg-primary text-white'
								: 'border-border bg-bg text-text-secondary hover:border-primary hover:text-primary'}"
					>
						{level}
					</button>
				{/each}
			</div>
		</section>
	</div>
</div>
