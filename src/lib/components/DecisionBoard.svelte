<script lang="ts">
	import { fly, fade } from 'svelte/transition';
	import type { SupabaseClient, User } from '@supabase/supabase-js';

	interface Assumption {
		text: string;
		confidence: 'high' | 'medium' | 'low';
		test: string;
		risk: string;
	}

	type FeedbackPriority = 'high' | 'medium';
	type FeedbackSeverity = 'critical' | 'improvement';

	interface FeedbackItem {
		issue: string;
		action: string;
		priority: FeedbackPriority;
		severity: FeedbackSeverity;
		category: 'problem-clarity' | 'assumptions' | 'evidence-support' | 'decision-clarity';
	}

	let { supabase, user }: { supabase: SupabaseClient; user: User | null } = $props();

	let problem = $state('');
	let assumptions = $state<Assumption[]>([
		{ text: '', confidence: 'medium', test: '', risk: '' },
		{ text: '', confidence: 'medium', test: '', risk: '' },
		{ text: '', confidence: 'medium', test: '', risk: '' }
	]);
	let evidence = $state('');
	let options = $state<string[]>(['', '', '']);
	let decision = $state('');
	let owner = $state('');
	let date = $state('');
	let status = $state<'Proposed' | 'Validated' | 'Invalidated'>('Proposed');
	let saving = $state(false);
	let message = $state('');
	let evaluation = $state<{ score: number; feedback: FeedbackItem[] } | null>(null);
	let actionRotation = $state<Record<string, number>>({});

	const severityRank: Record<FeedbackSeverity, number> = {
		critical: 0,
		improvement: 1
	};

	const priorityRank: Record<FeedbackPriority, number> = {
		high: 0,
		medium: 1
	};

	function nextAction(actionKey: string, variations: string[]) {
		const currentIndex = actionRotation[actionKey] ?? 0;
		actionRotation = { ...actionRotation, [actionKey]: currentIndex + 1 };
		return variations[currentIndex % variations.length];
	}

	function evaluateBoard() {
		let score = 0;
		const feedbackByCategory = new Map<FeedbackItem['category'], FeedbackItem>();

		function upsertFeedback(item: FeedbackItem) {
			const current = feedbackByCategory.get(item.category);
			if (!current) {
				feedbackByCategory.set(item.category, item);
				return;
			}

			const currentSeverity = severityRank[current.severity];
			const incomingSeverity = severityRank[item.severity];
			if (incomingSeverity < currentSeverity) {
				feedbackByCategory.set(item.category, item);
				return;
			}

			const currentPriority = priorityRank[current.priority];
			const incomingPriority = priorityRank[item.priority];
			if (incomingPriority < currentPriority) {
				feedbackByCategory.set(item.category, item);
			}
		}

		const hasProblem = problem.trim().length > 20;
		const hasDecision = decision.trim().length > 20;
		const hasEvidence = evidence.trim().length > 20;

		const nonEmptyAssumptions = assumptions.filter((a) => a.text.trim());
		const hasAssumptions = nonEmptyAssumptions.length > 0;
		const hasStrongAssumptions = nonEmptyAssumptions.length >= 2;
		const hasAssumptionTests = nonEmptyAssumptions.some((a) => a.test.trim().length > 0);
		const hasAssumptionRisks = nonEmptyAssumptions.some((a) => a.risk.trim().length > 0);

		if (hasProblem) score++;
		else
			upsertFeedback({
				issue: 'The problem statement is still unclear',
				action: nextAction('problem-unclear', [
					'Define the problem in one specific sentence, including who is affected',
					'Clarify the exact user outcome that is currently failing',
					'Reframe this as a concrete problem with scope and impact'
				]),
				priority: 'high',
				severity: 'critical',
				category: 'problem-clarity'
			});

		if (hasStrongAssumptions) score++;
		else
			upsertFeedback({
				issue: 'Key assumptions are underdefined',
				action: nextAction('assumptions-weak', [
					'Define the key assumptions this decision depends on',
					'Identify the 2-3 assumptions that would invalidate this decision if wrong',
					'Make the core assumptions explicit before moving forward'
				]),
				priority: 'medium',
				severity: 'improvement',
				category: 'assumptions'
			});

		if (hasEvidence) score++;

		if (hasDecision) score++;
		else
			upsertFeedback({
				issue: 'The decision is still unclear',
				action: nextAction('decision-unclear', [
					'State the decision clearly, including what will happen next',
					'Define one clear direction and the tradeoff being accepted',
					'Write the final decision as an explicit commitment'
				]),
				priority: 'high',
				severity: 'critical',
				category: 'decision-clarity'
			});

		if (hasAssumptions && !hasAssumptionTests)
			upsertFeedback({
				issue: 'Assumptions are not yet validated',
				action: nextAction('assumptions-no-test', [
					'Define how each key assumption will be tested',
					'Attach a concrete validation method to each major assumption',
					'Specify what evidence would confirm or invalidate each assumption'
				]),
				priority: 'high',
				severity: 'improvement',
				category: 'assumptions'
			});

		if (hasDecision && !hasEvidence)
			upsertFeedback({
				issue: 'This decision is not supported by strong evidence',
				action: 'Add concrete user signals such as quotes, metrics, or repeated behavior patterns',
				priority: 'high',
				severity: 'critical',
				category: 'evidence-support'
			});
		else if (!hasDecision && !hasEvidence)
			upsertFeedback({
				issue: 'Evidence is still too weak to support a decision',
				action: nextAction('evidence-weak-no-decision', [
					'Add concrete user signals such as quotes, metrics, or repeated behavior patterns',
					'Include observed behavior that clarifies what decision should be made',
					'Bring in specific user or market evidence before deciding'
				]),
				priority: 'medium',
				severity: 'improvement',
				category: 'evidence-support'
			});

		if (hasEvidence && !hasDecision)
			upsertFeedback({
				issue: 'Evidence has not been translated into a decision',
				action: nextAction('evidence-no-decision', [
					'Use the evidence to define a clear direction',
					'Convert the strongest evidence into a specific decision',
					'Derive one explicit decision from the current evidence'
				]),
				priority: 'high',
				severity: 'critical',
				category: 'decision-clarity'
			});

		if (hasAssumptions && !hasAssumptionRisks)
			upsertFeedback({
				issue: 'Risks are not yet considered',
				action: nextAction('assumptions-no-risk', [
					'Define what happens if the key assumptions are wrong',
					'Document the downside scenario for each critical assumption',
					'Clarify the primary risk if this assumption set fails'
				]),
				priority: 'medium',
				severity: 'improvement',
				category: 'assumptions'
			});

		const prioritizedFeedback = Array.from(feedbackByCategory.values())
			.sort((a, b) => {
				const severityDiff = severityRank[a.severity] - severityRank[b.severity];
				if (severityDiff !== 0) return severityDiff;
				return priorityRank[a.priority] - priorityRank[b.priority];
			})
			.slice(0, 3);

		return { score, feedback: prioritizedFeedback };
	}

	function clarityLevel(score: number) {
		return score <= 1 ? 'Weak' : score <= 3 ? 'Medium' : 'Strong';
	}

	function generateSummary() {
		const filledAssumptions = assumptions.filter((a) => a.text.trim());
		const riskSummary = filledAssumptions
			.filter((a) => a.risk.trim())
			.map((a) => a.risk)
			.join('; ');

		const summary = `We are solving: ${problem}

We believe:
${filledAssumptions.map((a) => `- ${a.text}`).join('\n')}

Based on:
${evidence}

We choose:
${decision}

If wrong:
${riskSummary || 'No risks documented'}`;

		return summary;
	}

	function copySummary() {
		const summary = generateSummary();
		navigator.clipboard.writeText(summary);
		message = 'Summary copied to clipboard!';
		setTimeout(() => (message = ''), 2000);
	}

	function addAssumption() {
		if (assumptions.length < 5) {
			assumptions = [...assumptions, { text: '', confidence: 'medium', test: '', risk: '' }];
		}
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

		// 1. Insert into decision_boards with new fields
		const { data: board, error: boardError } = await supabase
			.from('decision_boards')
			.insert({ problem, evidence, decision, owner, date, status, user_id: user?.id })
			.select()
			.single();

		if (boardError) {
			message = 'Error saving board: ' + boardError.message;
			saving = false;
			return;
		}

		const boardId = board.id;

		// 2. Insert assumptions (filter out empty ones)
		const nonEmptyAssumptions = assumptions.filter((a) => a.text.trim());
		if (nonEmptyAssumptions.length > 0) {
			const { error: assError } = await supabase
				.from('assumptions')
				.insert(
					nonEmptyAssumptions.map((assumption) => ({
						board_id: boardId,
						content: assumption.text,
						confidence: assumption.confidence,
						test: assumption.test,
						risk: assumption.risk
					}))
				);

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
			<p class="mb-4 rounded-lg bg-primary-soft px-3 py-2 text-xs text-text-secondary">What assumption, if wrong, breaks this?</p>
			<div class="flex flex-col gap-4">
				{#each assumptions as assumption, i (i)}
					<div class="rounded-lg border border-border bg-bg p-4" transition:fly={{ y: -8, duration: 200 }}>
						<div class="mb-3 flex items-start gap-2">
							<input
								type="text"
								bind:value={assumption.text}
								placeholder="What do we believe is true?"
								class="flex-1 rounded-lg border border-border bg-surface px-3 py-2 text-sm text-text placeholder-text-secondary outline-none transition-shadow focus:ring-2 focus:ring-primary-soft focus:border-primary"
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

						<div class="grid grid-cols-2 gap-3">
							<div>
								<label for="confidence-{i}" class="mb-1 block text-xs font-medium text-text-secondary">Confidence</label>
								<select
									id="confidence-{i}"
									bind:value={assumption.confidence}
									class="w-full rounded-lg border border-border bg-surface px-3 py-2 text-sm text-text outline-none transition-shadow focus:ring-2 focus:ring-primary-soft focus:border-primary"
								>
									<option value="low">Low</option>
									<option value="medium">Medium</option>
									<option value="high">High</option>
								</select>
							</div>
							<div>
								<label for="test-{i}" class="mb-1 block text-xs font-medium text-text-secondary">How will we test?</label>
								<input
									id="test-{i}"
									type="text"
									bind:value={assumption.test}
									placeholder="Test method..."
									class="w-full rounded-lg border border-border bg-surface px-3 py-2 text-sm text-text placeholder-text-secondary outline-none transition-shadow focus:ring-2 focus:ring-primary-soft focus:border-primary"
								/>
							</div>
						</div>

						<div class="mt-3">
							<label for="risk-{i}" class="mb-1 block text-xs font-medium text-text-secondary">What if wrong?</label>
							<input
								id="risk-{i}"
								type="text"
								bind:value={assumption.risk}
								placeholder="Consequence if incorrect..."
								class="w-full rounded-lg border border-border bg-surface px-3 py-2 text-sm text-text placeholder-text-secondary outline-none transition-shadow focus:ring-2 focus:ring-primary-soft focus:border-primary"
							/>
						</div>
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

		<!-- Decision Details (Accountability) -->
		<section class="rounded-2xl border border-border bg-surface p-6 shadow-sm">
			<h2 class="mb-4 block text-sm font-medium text-text">Decision Details</h2>
			<div class="grid grid-cols-3 gap-4">
				<div>
					<label for="owner" class="mb-1 block text-xs font-medium text-text-secondary">Owner</label>
					<input
						id="owner"
						type="text"
						bind:value={owner}
						placeholder="Who is accountable?"
						class="w-full rounded-lg border border-border bg-bg px-3 py-2 text-sm text-text placeholder-text-secondary outline-none transition-shadow focus:ring-2 focus:ring-primary-soft focus:border-primary"
					/>
				</div>
				<div>
					<label for="date" class="mb-1 block text-xs font-medium text-text-secondary">Date</label>
					<input
						id="date"
						type="date"
						bind:value={date}
						class="w-full rounded-lg border border-border bg-bg px-3 py-2 text-sm text-text outline-none transition-shadow focus:ring-2 focus:ring-primary-soft focus:border-primary"
					/>
				</div>
				<div>
					<label for="status" class="mb-1 block text-xs font-medium text-text-secondary">Status</label>
					<select
						id="status"
						bind:value={status}
						class="w-full rounded-lg border border-border bg-bg px-3 py-2 text-sm text-text outline-none transition-shadow focus:ring-2 focus:ring-primary-soft focus:border-primary"
					>
						<option value="Proposed">Proposed</option>
						<option value="Validated">Validated</option>
						<option value="Invalidated">Invalidated</option>
					</select>
				</div>
			</div>
		</section>

		<!-- Decision Summary -->
		<section class="rounded-2xl border border-border bg-linear-to-br from-primary-soft to-bg p-6 shadow-sm" transition:fade={{ duration: 200 }}>
			<div class="mb-4 flex items-center justify-between">
				<h2 class="text-sm font-medium text-text">Decision Summary</h2>
				<button
					onclick={copySummary}
					class="flex items-center gap-1.5 rounded-lg px-3 py-1.5 text-xs font-medium text-primary transition-colors hover:bg-primary hover:text-white"
				>
					<svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M16 4h2a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H6a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2h2"/><rect x="8" y="2" width="8" height="4" rx="1" ry="1"/></svg>
					Copy
				</button>
			</div>
			<pre class="whitespace-pre-wrap rounded-lg border border-border bg-surface p-4 text-xs leading-relaxed text-text"><code>{generateSummary()}</code></pre>
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
				<span class="rounded-lg px-2.5 py-1 text-xs font-medium {clarityLevel(evaluation.score) === 'Strong' ? 'bg-blue-100 text-blue-700' : clarityLevel(evaluation.score) === 'Medium' ? 'bg-primary-soft text-primary' : 'bg-gray-100 text-text-secondary'}">
					{clarityLevel(evaluation.score)}
				</span>
			</div>
			{#if evaluation.feedback.length > 0}
				<p class="mt-3 text-sm font-medium text-text">Decision review</p>
				<p class="mt-1 text-xs text-text-secondary">Focus on these first to strengthen this decision.</p>
				<ul class="mt-4 flex flex-col gap-4">
					{#each evaluation.feedback as item}
						<li class="rounded-lg border border-border bg-bg px-3 py-3">
							<div class="flex items-center justify-between gap-3">
								<p class="text-sm {item.severity === 'critical' ? 'font-semibold text-text' : 'font-medium text-text'}">{item.issue}</p>
								<span class="rounded-md border px-2 py-0.5 text-[11px] uppercase tracking-wide {item.severity === 'critical' ? 'border-red-200 text-red-700 bg-red-50/50' : 'border-border text-text-secondary bg-surface'}">
									{item.severity === 'critical' ? 'Critical' : 'Improve'}
								</span>
							</div>
							<p class="mt-2 text-xs text-text-secondary">-> {item.action}</p>
						</li>
					{/each}
				</ul>
			{:else}
				<p class="mt-4 text-sm font-semibold text-text">Clarity: Strong</p>
				<p class="mt-2 text-sm text-text">This decision is clear and well-supported.</p>
				<p class="mt-1 text-sm text-text-secondary">Continue validating with real-world evidence as you execute.</p>
			{/if}
		</div>
	{/if}
</div>
