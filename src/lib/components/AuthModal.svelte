<script lang="ts">
	import type { SupabaseClient } from '@supabase/supabase-js';

	type AuthMode = 'signin' | 'signup';

	interface AuthModalProps {
		open: boolean;
		mode?: AuthMode;
		supabase: SupabaseClient;
		onClose?: () => void;
		onSuccess?: () => void | Promise<void>;
	}

	let {
		open,
		mode = 'signin',
		supabase,
		onClose = () => {},
		onSuccess = () => {}
	}: AuthModalProps = $props();

	let activeTab = $state<AuthMode>('signin');
	let email = $state('');
	let password = $state('');
	let loading = $state(false);
	let message = $state('');

	$effect(() => {
		if (open) {
			activeTab = mode;
			message = '';
		}
	});

	function closeModal() {
		onClose();
	}

	function handleBackdropClick(event: MouseEvent) {
		if (event.target === event.currentTarget) {
			closeModal();
		}
	}

	function handleKeydown(event: KeyboardEvent) {
		if (event.key === 'Escape') {
			closeModal();
		}
	}

	async function handleSubmit(event: SubmitEvent) {
		event.preventDefault();
		loading = true;
		message = '';

		if (activeTab === 'signin') {
			const { error } = await supabase.auth.signInWithPassword({ email, password });
			if (error) {
				message = error.message;
				loading = false;
				return;
			}

			await onSuccess();
			loading = false;
			return;
		}

		const { error } = await supabase.auth.signUp({ email, password });
		if (error) {
			message = error.message;
			loading = false;
			return;
		}

		message = 'Check your email for a confirmation link.';
		loading = false;
	}
</script>

{#if open}
	<div
		class="fixed inset-0 z-50 flex items-center justify-center bg-black/30 p-4"
		onclick={handleBackdropClick}
		onkeydown={handleKeydown}
		tabindex="-1"
		role="button"
		aria-label="Close authentication modal"
	>
		<div class="w-full max-w-md rounded-2xl border border-border bg-surface p-6 shadow-sm" role="dialog" aria-modal="true" aria-labelledby="auth-title">
			<div class="mb-5 flex items-start justify-between">
				<div>
					<h2 id="auth-title" class="text-2xl font-semibold tracking-tight text-text">{activeTab === 'signin' ? 'Sign in' : 'Get access'}</h2>
					<p class="mt-1 text-sm text-text-secondary">Use Product Signal from the same workspace.</p>
				</div>
				<button
					onclick={closeModal}
					class="cursor-pointer rounded-lg px-2 py-1 text-sm text-text-secondary transition-colors hover:bg-primary-soft hover:text-primary"
					aria-label="Close"
				>
					Close
				</button>
			</div>

			<div class="mb-5 grid grid-cols-2 rounded-xl border border-border bg-bg p-1">
				<button
					type="button"
					onclick={() => {
						activeTab = 'signin';
						message = '';
					}}
					class="cursor-pointer rounded-lg px-3 py-2 text-sm font-medium transition-colors {activeTab === 'signin'
						? 'bg-surface text-text shadow-sm'
						: 'text-text-secondary hover:text-text'}"
				>
					Sign in
				</button>
				<button
					type="button"
					onclick={() => {
						activeTab = 'signup';
						message = '';
					}}
					class="cursor-pointer rounded-lg px-3 py-2 text-sm font-medium transition-colors {activeTab === 'signup'
						? 'bg-surface text-text shadow-sm'
						: 'text-text-secondary hover:text-text'}"
				>
					Sign up
				</button>
			</div>

			<form onsubmit={handleSubmit} class="space-y-4">
				<div>
					<label for="auth-email" class="mb-1 block text-sm font-medium text-text">Email</label>
					<input
						id="auth-email"
						type="email"
						bind:value={email}
						required
						placeholder="you@example.com"
						class="w-full rounded-xl border border-border bg-bg px-4 py-2.5 text-sm text-text placeholder-text-secondary outline-none transition-shadow focus:border-primary focus:ring-2 focus:ring-primary-soft"
					/>
				</div>

				<div>
					<label for="auth-password" class="mb-1 block text-sm font-medium text-text">Password</label>
					<input
						id="auth-password"
						type="password"
						bind:value={password}
						required
						minlength={6}
						placeholder="Enter password"
						class="w-full rounded-xl border border-border bg-bg px-4 py-2.5 text-sm text-text placeholder-text-secondary outline-none transition-shadow focus:border-primary focus:ring-2 focus:ring-primary-soft"
					/>
				</div>

				{#if message}
					<p class="text-sm" class:text-red-500={!message.includes('Check your email')} class:text-green-600={message.includes('Check your email')}>
						{message}
					</p>
				{/if}

				<button
					type="submit"
					disabled={loading}
					class="w-full cursor-pointer rounded-xl bg-primary px-4 py-2.5 text-sm font-semibold text-white transition-colors hover:bg-primary-hover disabled:cursor-not-allowed disabled:opacity-60"
				>
					{loading ? 'Please wait...' : activeTab === 'signin' ? 'Sign in' : 'Create account'}
				</button>
			</form>
		</div>
	</div>
{/if}