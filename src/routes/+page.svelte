<script lang="ts">
	import { invalidate } from '$app/navigation';
	import DecisionBoard from '$lib/components/DecisionBoard.svelte';
	import AuthModal from '$lib/components/AuthModal.svelte';

	let { data } = $props();

	type AuthMode = 'signin' | 'signup';

	let authOpen = $state(false);
	let authMode = $state<AuthMode>('signup');

	const problemPoints = [
		'Teams ship unclear decisions',
		"Research doesn't change anything",
		"Everything feels 'fine' but doesn't work"
	];

	const systemSteps = [
		{
			title: 'Define the problem',
			description: 'Write one concrete problem with scope, owner, and impact.'
		},
		{
			title: 'Make assumptions explicit',
			description: 'List what must be true for this direction to work.'
		},
		{
			title: 'Validate them',
			description: 'Attach evidence and test plans before committing resources.'
		},
		{
			title: 'Make a clear decision',
			description: 'Capture the tradeoff, owner, and next action in one summary.'
		}
	];

	const features = [
		{
			title: 'Decision Board',
			description: 'Structure decisions clearly and keep context in one place.'
		},
		{
			title: 'Research Distiller',
			description: 'Turn raw interviews, notes, and signals into usable insight.'
		},
		{
			title: 'Clarity Feedback',
			description: 'Identify weak thinking before weak decisions become roadmap work.'
		},
		{
			title: 'Decision Summary',
			description: 'Produce clear, shareable output for teams and stakeholders.'
		}
	];

	const credibilityPoints = ['Built for founders and PMs', 'Single source of decision truth', 'Early access: limited seats'];

	function openAuth(mode: AuthMode) {
		authMode = mode;
		authOpen = true;
	}

	function closeAuth() {
		authOpen = false;
	}

	async function handleAuthSuccess() {
		authOpen = false;
		await invalidate('supabase:auth');
	}
</script>

<svelte:head>
	<title>{data.user ? 'Product Signal — Workspace' : 'Product Signal — Make Better Product Decisions'}</title>
</svelte:head>

<div class="min-h-screen bg-bg text-text">
	{#if data.user}
		<main class="mx-auto w-full max-w-6xl px-4 pb-16 pt-8 sm:px-6 lg:px-8">
			<DecisionBoard supabase={data.supabase} user={data.user} />
		</main>
	{:else}
		<nav class="sticky top-0 z-20 border-b border-border bg-surface/95 backdrop-blur">
			<div class="mx-auto flex w-full max-w-6xl items-center justify-between px-4 py-4 sm:px-6 lg:px-8">
				<div class="flex items-center gap-3">
					<p class="text-base font-semibold tracking-tight text-text">Product Signal</p>
					<span class="hidden rounded-full border border-border bg-bg px-2.5 py-1 text-xs font-medium text-text-secondary sm:inline">Early access</span>
				</div>
				<div class="flex items-center gap-2 sm:gap-3">
					<button
						onclick={() => openAuth('signin')}
						class="cursor-pointer rounded-xl px-3 py-2 text-sm font-medium text-text-secondary transition-colors hover:bg-primary-soft hover:text-primary"
					>
						Sign in
					</button>
					<button
						onclick={() => openAuth('signup')}
						class="cursor-pointer rounded-xl bg-primary px-4 py-2 text-sm font-semibold text-white transition-colors hover:bg-primary-hover"
					>
						Get access
					</button>
				</div>
			</div>
		</nav>

		<main class="mx-auto w-full max-w-6xl px-4 pb-24 pt-8 sm:px-6 sm:pt-12 lg:px-8 lg:pt-16">
			<section class="grid items-start gap-8 lg:grid-cols-[minmax(0,1.05fr)_minmax(0,0.95fr)] lg:gap-10">
				<div class="space-y-7">
					<p class="inline-flex rounded-full border border-border bg-surface px-3 py-1.5 text-xs font-medium uppercase tracking-[0.14em] text-text-secondary">For product teams and founders</p>
					<h1 class="max-w-2xl text-4xl font-semibold leading-tight tracking-tight text-text sm:text-5xl sm:leading-tight">
						Make better product decisions.
						<br class="hidden sm:block" />
						Not faster ones.
					</h1>
					<p class="max-w-xl text-lg leading-relaxed text-text-secondary">
						Structure your thinking, test assumptions, and get clear before you build.
					</p>
					<div class="flex flex-wrap items-center gap-3">
						<button
							onclick={() => openAuth('signup')}
							class="cursor-pointer rounded-xl bg-primary px-5 py-3 text-sm font-semibold text-white transition-colors hover:bg-primary-hover"
						>
							Get Product Signal
						</button>
						<button
							onclick={() => openAuth('signin')}
							class="cursor-pointer rounded-xl border border-border bg-surface px-5 py-3 text-sm font-semibold text-text transition-colors hover:bg-primary-soft"
						>
							Sign in
						</button>
					</div>
					<ul class="flex flex-wrap items-center gap-x-3 gap-y-2 text-xs font-medium text-text-secondary">
						{#each credibilityPoints as item, index}
							<li class="inline-flex items-center gap-3">
								<span>{item}</span>
								{#if index < credibilityPoints.length - 1}
									<span class="hidden h-1 w-1 rounded-full bg-border sm:inline" aria-hidden="true"></span>
								{/if}
							</li>
						{/each}
					</ul>
				</div>

				<div class="rounded-2xl border border-border bg-surface p-3 shadow-sm">
					<div class="mb-2 flex items-center justify-between rounded-xl border border-border bg-bg px-3 py-2">
						<p class="text-xs font-medium uppercase tracking-[0.12em] text-text-secondary">Live Product View</p>
						<span class="rounded-full bg-primary-soft px-2 py-0.5 text-xs font-semibold text-primary">Decision Board</span>
					</div>
					<div class="max-h-[35rem] overflow-hidden rounded-xl border border-border bg-bg" aria-hidden="true">
						<div class="pointer-events-none origin-top-left scale-[0.84] sm:scale-[0.9]">
							<DecisionBoard supabase={data.supabase} user={data.user} />
						</div>
					</div>
				</div>
			</section>

			<section class="mt-20 border-t border-border pt-14 sm:mt-24 sm:pt-16">
				<div class="grid gap-8 lg:grid-cols-[minmax(0,0.9fr)_minmax(0,1.1fr)]">
					<div>
						<p class="text-xs font-medium uppercase tracking-[0.12em] text-text-secondary">The problem</p>
						<h2 class="mt-3 text-2xl font-semibold tracking-tight text-text sm:text-3xl">Most teams move fast through unclear decisions.</h2>
					</div>
					<div class="grid gap-4 sm:grid-cols-3">
						{#each problemPoints as point, index}
							<div class="rounded-2xl border border-border bg-surface p-5">
								<p class="text-xs font-semibold uppercase tracking-[0.12em] text-text-secondary">0{index + 1}</p>
								<p class="mt-3 text-base leading-relaxed text-text">{point}</p>
							</div>
						{/each}
					</div>
				</div>
			</section>

			<section class="mt-20 border-t border-border pt-14 sm:mt-24 sm:pt-16">
				<div class="flex items-end justify-between gap-4">
					<div>
						<p class="text-xs font-medium uppercase tracking-[0.12em] text-text-secondary">The system</p>
						<h2 class="mt-3 text-2xl font-semibold tracking-tight text-text sm:text-3xl">A simple sequence for stronger decisions.</h2>
					</div>
				</div>
				<ol class="mt-6 grid gap-4 sm:grid-cols-2">
					{#each systemSteps as step, index}
						<li class="rounded-2xl border border-border bg-surface p-5">
							<p class="text-xs font-semibold uppercase tracking-[0.12em] text-text-secondary">Step {index + 1}</p>
							<p class="mt-2 text-lg font-semibold text-text">{step.title}</p>
							<p class="mt-2 text-sm leading-relaxed text-text-secondary">{step.description}</p>
						</li>
					{/each}
				</ol>
			</section>

			<section class="mt-20 border-t border-border pt-14 sm:mt-24 sm:pt-16">
				<p class="text-xs font-medium uppercase tracking-[0.12em] text-text-secondary">Features</p>
				<h2 class="mt-3 text-2xl font-semibold tracking-tight text-text sm:text-3xl">Built for decision quality, not dashboard noise.</h2>
				<div class="mt-6 grid gap-4 sm:grid-cols-2">
					{#each features as feature}
						<article class="rounded-2xl border border-border bg-surface p-5">
							<div class="flex items-start justify-between gap-3">
								<h3 class="text-base font-semibold text-text">{feature.title}</h3>
								<span class="text-sm font-semibold text-primary">→</span>
							</div>
							<p class="mt-2 text-sm leading-relaxed text-text-secondary">{feature.description}</p>
						</article>
					{/each}
				</div>
			</section>

			<section class="mt-20 border-t border-border pt-14 sm:mt-24 sm:pt-16">
				<p class="text-xs font-medium uppercase tracking-[0.12em] text-text-secondary">Output</p>
				<h2 class="mt-3 text-2xl font-semibold tracking-tight text-text sm:text-3xl">Decision summary your team can align on.</h2>
				<div class="mt-6 rounded-2xl border border-border bg-primary-soft/40 p-6 sm:p-8">
					<div class="grid gap-6 lg:grid-cols-[minmax(0,1fr)_16rem]">
						<div class="rounded-2xl border border-border bg-surface p-6">
							<p class="text-xs font-medium uppercase tracking-wide text-text-secondary">Decision Summary</p>
							<div class="mt-4 space-y-4 text-sm leading-relaxed text-text">
								<p><span class="font-semibold">We are solving:</span> Activation drops after first session because onboarding assumes prior workflow knowledge.</p>
								<p><span class="font-semibold">We believe:</span></p>
								<ul class="list-disc space-y-1 pl-5 text-text-secondary">
									<li>Users need a guided first-win path in less than 10 minutes.</li>
									<li>Current setup language hides the core action.</li>
									<li>Activation increases when next steps are explicit.</li>
								</ul>
								<p><span class="font-semibold">Based on:</span> 14 interviews, drop-off data from onboarding step 2, and support tickets about unclear setup.</p>
								<p><span class="font-semibold">We choose:</span> Replace generic onboarding with a role-based guided path and measured milestone checks.</p>
								<p><span class="font-semibold">If wrong:</span> We lose 2 weeks and revert to the current flow with no migration risk.</p>
							</div>
						</div>
						<aside class="rounded-2xl border border-border bg-surface p-5">
							<p class="text-xs font-semibold uppercase tracking-[0.12em] text-text-secondary">Why it works</p>
							<ul class="mt-3 space-y-3 text-sm text-text-secondary">
								<li>Clear problem framing</li>
								<li>Explicit assumptions</li>
								<li>Evidence-linked decision</li>
								<li>Owner and tradeoff clarity</li>
							</ul>
						</aside>
					</div>
				</div>
			</section>

			<section class="mt-20 border-t border-border pt-14 sm:mt-24 sm:pt-16">
				<p class="text-xs font-medium uppercase tracking-[0.12em] text-text-secondary">Pricing</p>
				<div class="mt-4 max-w-xl rounded-2xl border border-border bg-surface p-6 sm:p-8">
					<p class="text-sm text-text-secondary">Early access</p>
					<p class="mt-2 text-4xl font-semibold tracking-tight text-text">$75 lifetime access</p>
					<p class="mt-2 text-sm text-text-secondary">Limited spots</p>
					<ul class="mt-5 space-y-2 text-sm text-text-secondary">
						<li>Decision Board</li>
						<li>Research Distiller</li>
						<li>Clarity Feedback</li>
						<li>Decision Summary export</li>
					</ul>
					<button
						onclick={() => openAuth('signup')}
						class="mt-6 cursor-pointer rounded-xl bg-primary px-5 py-3 text-sm font-semibold text-white transition-colors hover:bg-primary-hover"
					>
						Get access
					</button>
				</div>
			</section>

			<section class="mt-20 rounded-2xl border border-border bg-surface p-8 sm:mt-24 sm:p-10">
				<p class="max-w-3xl text-2xl font-semibold tracking-tight text-text sm:text-3xl">
					Most product decisions are guesses.
					<br />
					Get signal before you build.
				</p>
				<div class="mt-6 flex flex-wrap items-center gap-3">
					<button
						onclick={() => openAuth('signup')}
						class="cursor-pointer rounded-xl bg-primary px-5 py-3 text-sm font-semibold text-white transition-colors hover:bg-primary-hover"
					>
						Get Product Signal
					</button>
					<button
						onclick={() => openAuth('signin')}
						class="cursor-pointer rounded-xl border border-border bg-bg px-5 py-3 text-sm font-semibold text-text transition-colors hover:bg-primary-soft"
					>
						Sign in
					</button>
				</div>
			</section>
		</main>

		<div class="fixed inset-x-0 bottom-0 z-30 border-t border-border bg-surface/95 p-3 backdrop-blur sm:hidden">
			<button
				onclick={() => openAuth('signup')}
				class="w-full cursor-pointer rounded-xl bg-primary px-5 py-3 text-sm font-semibold text-white transition-colors hover:bg-primary-hover"
			>
				Get Product Signal
			</button>
		</div>

		<AuthModal
			open={authOpen}
			mode={authMode}
			supabase={data.supabase}
			onClose={closeAuth}
			onSuccess={handleAuthSuccess}
		/>
	{/if}
</div>
