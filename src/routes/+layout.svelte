<script lang="ts">
	import '../app.css';
	import Header from '../lib/components/header.svelte';
	let { children } = $props();
</script>



<div class="bg-stone-900 text-amber-50 min-h-screen	">
	<Header/>
	<main class="mt-5">
		{@render children()}
	</main>
	
</div>

<style>
	main{
		width: 80%;
		justify-self: center;
	}
</style>