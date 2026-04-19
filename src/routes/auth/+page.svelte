<script lang="ts">
	import { goto } from '$app/navigation';

	let { data } = $props();

	let email = $state('');
	let password = $state('');
	let isLogin = $state(true);
	let loading = $state(false);
	let message = $state('');

	async function handleSubmit() {
		loading = true;
		message = '';

		if (isLogin) {
			const { error } = await data.supabase.auth.signInWithPassword({ email, password });
			if (error) {
				message = error.message;
				loading = false;
				return;
			}
			goto('/');
		} else {
			const { error } = await data.supabase.auth.signUp({ email, password });
			if (error) {
				message = error.message;
				loading = false;
				return;
			}
			message = 'Check your email for a confirmation link!';
		}

		loading = false;
	}
</script>

<svelte:head>
	<title>Product Signal — {isLogin ? 'Log In' : 'Sign Up'}</title>
</svelte:head>

<div class="flex min-h-screen items-center justify-center bg-bg px-4">
	<div class="w-full max-w-sm rounded-2xl border border-border bg-surface p-8 shadow-sm">
		<h1 class="mb-6 text-2xl font-semibold tracking-tight text-text">
			{isLogin ? 'Log In' : 'Sign Up'}
		</h1>

		<form onsubmit={handleSubmit} class="flex flex-col gap-4">
			<div>
				<label for="email" class="mb-1 block text-sm font-medium text-text">Email</label>
				<input
					id="email"
					type="email"
					bind:value={email}
					required
					placeholder="you@example.com"
					class="w-full rounded-xl border border-border bg-bg px-4 py-2.5 text-sm text-text placeholder-text-secondary outline-none transition-shadow focus:ring-2 focus:ring-primary-soft focus:border-primary"
				/>
			</div>

			<div>
				<label for="password" class="mb-1 block text-sm font-medium text-text">Password</label>
				<input
					id="password"
					type="password"
					bind:value={password}
					required
					minlength={6}
					placeholder="••••••••"
					class="w-full rounded-xl border border-border bg-bg px-4 py-2.5 text-sm text-text placeholder-text-secondary outline-none transition-shadow focus:ring-2 focus:ring-primary-soft focus:border-primary"
				/>
			</div>

			{#if message}
				<p class="text-sm" class:text-red-500={!message.includes('Check')} class:text-green-600={message.includes('Check')}>{message}</p>
			{/if}

			<button
				type="submit"
				disabled={loading}
				class="mt-2 rounded-xl bg-primary px-4 py-2.5 text-sm font-semibold text-white shadow transition-colors hover:bg-primary-hover disabled:opacity-50"
			>
				{loading ? 'Please wait...' : isLogin ? 'Log In' : 'Sign Up'}
			</button>
		</form>

		<p class="mt-4 text-center text-sm text-text-secondary">
			{isLogin ? "Don't have an account?" : 'Already have an account?'}
			<button
				onclick={() => { isLogin = !isLogin; message = ''; }}
				class="font-medium text-primary hover:underline cursor-pointer"
			>
				{isLogin ? 'Sign Up' : 'Log In'}
			</button>
		</p>
	</div>
</div>
