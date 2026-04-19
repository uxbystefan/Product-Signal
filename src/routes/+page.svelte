<script lang="ts">
	import DecisionBoard from '$lib/components/DecisionBoard.svelte';

	let { data } = $props();
</script>

<svelte:head>
	<title>Product Signal — Decision Board</title>
</svelte:head>

<div class="min-h-screen bg-bg">
	<DecisionBoard supabase={data.supabase} user={data.user} />
</div>
