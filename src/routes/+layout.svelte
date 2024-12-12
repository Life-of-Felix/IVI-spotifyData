<script lang="ts">
	import '../app.css';
	import Header from '../lib/components/header.svelte';
	import Footer from '../lib/components/footer.svelte';	
	let { children } = $props();
</script>



<div class="bg-stone-900 text-amber-50">
	<Header/>
	<main class="mt-5 min-h-screen">
		{@render children()}
	</main>
	<Footer/>
</div>

<style>
	main{
		width: 80%;
		justify-self: center;
	}
</style>