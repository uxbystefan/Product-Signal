<script lang="ts">
	import '../app.css';
	import favicon from '$lib/assets/favicon.svg';
	import { invalidate } from '$app/navigation';
	import { goto } from '$app/navigation';
	import { onMount } from 'svelte';

	let { data, children } = $props();

	onMount(() => {
		const { data: { subscription } } = data.supabase.auth.onAuthStateChange((_, session) => {
			if (session?.expires_at !== data.session?.expires_at) {
				invalidate('supabase:auth');
			}
		});

		return () => subscription.unsubscribe();
	});

	async function logout() {
		await data.supabase.auth.signOut();
		goto('/');
	}
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
</svelte:head>

{#if data.session}
	<header class="flex items-center justify-end gap-4 px-6 py-3 border-b border-border bg-surface">
		<span class="text-sm text-text-secondary">{data.session.user.email}</span>
		<button
			onclick={logout}
			class="rounded-lg px-3 py-1.5 text-sm font-medium text-text-secondary transition-colors hover:bg-primary-soft hover:text-primary cursor-pointer"
		>
			Log out
		</button>
	</header>
{/if}

{@render children()}
