<script>
	import '../app.css';

	let { children } = $props();
</script>

<div class="max-w-sm mx-auto min-h-screen bg-gray-100">
	{@render children()}
</div>

