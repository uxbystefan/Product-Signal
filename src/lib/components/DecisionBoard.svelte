<script lang="ts">
	import { fly, fade } from 'svelte/transition';
	import type { SupabaseClient, User } from '@supabase/supabase-js';

	let { supabase, user }: { supabase: SupabaseClient; user: User | null } = $props();

	let problem = $state('');
	let assumptions = $state<string[]>(['', '', '']);
	let evidence = $state('');
	let options = $state<string[]>(['', '', '']);
	let decision = $state('');
	let saving = $state(false);
	let message = $state('');
	let evaluation = $state<{ score: number; level: string; feedback: string[] } | null>(null);

	function evaluateBoard() {
		let score = 0;
		const feedback: string[] = [];

		if (problem.trim().length > 20) score++;
		else feedback.push('Problem is unclear');

		const filledAssumptions = assumptions.filter((a) => a.trim()).length;
		if (filledAssumptions >= 2) score++;
		else feedback.push('Add more assumptions');

		if (evidence.trim().length > 20) score++;
		else feedback.push('Evidence is weak');

		if (decision.trim().length > 20) score++;
		else feedback.push('Decision is not clearly defined');

		const level = score <= 1 ? 'Weak' : score <= 3 ? 'Medium' : 'Strong';

		return { score, level, feedback };
	}

	function addAssumption() {
		if (assumptions.length < 5) assumptions = [...assumptions, ''];
	}

	function removeAssumption(index: number) {
		if (assumptions.length > 3) assumptions = assumptions.filter((_, i) => i !== index);
	}

	function addOption() {
		if (options.length < 5) options = [...options, ''];
	}

	function removeOption(index: number) {
		if (options.length > 3) options = options.filter((_, i) => i !== index);
	}

	async function saveBoard() {
		saving = true;
		message = '';

		// 1. Insert into decision_boards
		const { data: board, error: boardError } = await supabase
			.from('decision_boards')
			.insert({ problem, evidence, decision, user_id: user?.id })
			.select()
			.single();

		if (boardError) {
			message = 'Error saving board: ' + boardError.message;
			saving = false;
			return;
		}

		const boardId = board.id;

		// 2. Insert assumptions (filter out empty ones)
		const nonEmptyAssumptions = assumptions.filter((a) => a.trim());
		if (nonEmptyAssumptions.length > 0) {
			const { error: assError } = await supabase
				.from('assumptions')
				.insert(nonEmptyAssumptions.map((content) => ({ board_id: boardId, content })));

			if (assError) {
				message = 'Board saved but error saving assumptions: ' + assError.message;
				saving = false;
				return;
			}
		}

		// 3. Insert options (filter out empty ones)
		const nonEmptyOptions = options.filter((o) => o.trim());
		if (nonEmptyOptions.length > 0) {
			const { error: optError } = await supabase
				.from('options')
				.insert(nonEmptyOptions.map((content) => ({ board_id: boardId, content })));

			if (optError) {
				message = 'Board saved but error saving options: ' + optError.message;
				saving = false;
				return;
			}
		}

		message = 'Decision saved successfully!';
		evaluation = evaluateBoard();
		saving = false;
	}
</script>

<div class="mx-auto max-w-2xl px-4 py-16">
	<h1 class="mb-10 text-3xl font-semibold tracking-tight text-text">Decision Board</h1>

	<div class="flex flex-col gap-6">
		<!-- Problem -->
		<section class="rounded-2xl border border-border bg-surface p-6 shadow-sm">
			<label for="problem" class="mb-2 block text-sm font-medium text-text">Problem</label>
			<p class="mb-2 rounded-lg bg-primary-soft px-3 py-2 text-xs text-text-secondary">Are we solving the right problem?</p>
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
			<p class="mb-2 rounded-lg bg-primary-soft px-3 py-2 text-xs text-text-secondary">What assumption, if wrong, breaks this?</p>
			<div class="flex flex-col gap-2">
				{#each assumptions as assumption, i (i)}
					<div class="flex items-center gap-2" transition:fly={{ y: -8, duration: 200 }}>
						<input
							type="text"
							bind:value={assumptions[i]}
							placeholder="What do we believe is true?"
							class="flex-1 rounded-xl border border-border bg-bg px-4 py-2.5 text-sm text-text placeholder-text-secondary outline-none transition-shadow focus:ring-2 focus:ring-primary-soft focus:border-primary"
						/>
						{#if assumptions.length > 3}
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
				disabled={assumptions.length >= 5}
				class="mt-3 cursor-pointer rounded-lg px-3 py-1.5 text-sm font-medium text-primary transition-colors hover:bg-primary-soft disabled:cursor-not-allowed disabled:opacity-40"
			>
				+ Add assumption
			</button>
		</section>

		<!-- Evidence -->
		<section class="rounded-2xl border border-border bg-surface p-6 shadow-sm">
			<label for="evidence" class="mb-2 block text-sm font-medium text-text">Evidence</label>
			<p class="mb-2 rounded-lg bg-primary-soft px-3 py-2 text-xs text-text-secondary">What evidence do we actually have?</p>
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
						{#if options.length > 3}
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
				disabled={options.length >= 5}
				class="mt-3 cursor-pointer rounded-lg px-3 py-1.5 text-sm font-medium text-primary transition-colors hover:bg-primary-soft disabled:cursor-not-allowed disabled:opacity-40"
			>
				+ Add option
			</button>
		</section>

		<!-- Decision -->
		<section class="rounded-2xl border border-border bg-surface p-6 shadow-sm">
			<label for="decision" class="mb-2 block text-sm font-medium text-text">Decision</label>
			<p class="mb-2 rounded-lg bg-primary-soft px-3 py-2 text-xs text-text-secondary">What are we avoiding deciding?</p>
			<textarea
				id="decision"
				bind:value={decision}
				placeholder="What are we choosing and why?"
				rows={3}
				class="w-full resize-none rounded-xl border border-border bg-bg px-4 py-3 text-sm text-text placeholder-text-secondary outline-none transition-shadow focus:ring-2 focus:ring-primary-soft focus:border-primary"
			></textarea>
		</section>
	</div>

	<div class="flex items-center justify-end gap-4 mt-8">
		{#if message}
			<p class="text-sm" class:text-green-600={message.includes('successfully')} class:text-red-500={!message.includes('successfully')}>{message}</p>
		{/if}
		<button
			onclick={saveBoard}
			disabled={saving}
			class="rounded-lg bg-primary px-6 py-2 text-white font-semibold shadow hover:bg-primary-dark transition-colors disabled:opacity-50"
		>
			{saving ? 'Saving...' : 'Save Decision'}
		</button>
	</div>

	{#if evaluation}
		<div class="mt-6 rounded-2xl border border-border bg-surface p-6 shadow-sm" transition:fade={{ duration: 200 }}>
			<div class="flex items-center gap-2">
				<h2 class="text-sm font-semibold text-text">Clarity:</h2>
				<span class="rounded-lg px-2.5 py-1 text-xs font-medium {evaluation.level === 'Strong' ? 'bg-blue-100 text-blue-700' : evaluation.level === 'Medium' ? 'bg-primary-soft text-primary' : 'bg-gray-100 text-text-secondary'}">
					{evaluation.level}
				</span>
			</div>
			{#if evaluation.feedback.length > 0}
				<ul class="mt-3 flex flex-col gap-1">
					{#each evaluation.feedback as item}
						<li class="text-sm text-text-secondary">• {item}</li>
					{/each}
				</ul>
			{/if}
		</div>
	{/if}
</div>
