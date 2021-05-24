<script>
	export let open = false;
	export let exportData;
	export let toggleHelp;
	export let toggleSearch;
</script>

<aside class="" class:open>
	<nav class="">
			<item on:click={toggleSearch}>Search</item>
			<item on:click={exportData}>Export</item>
			<item on:click={toggleHelp}>Help</item>
	</nav>
</aside>

<style>

	item {
		cursor: pointer; 
		display: block;
		color: white;
		transition: transform 0.3s;
		margin-bottom: 6px;
	}

	item:last-child{
		margin-bottom: 0;
	}

	item:hover{
		transform: scale(1.05);
	}

	aside {
		position: absolute;
		padding: 20px;
		font-size: 1.2rem;
		top: 61px;
		left: -100%;
		transition: left 0.3s ease-in-out;
		color: white;
	}
	
	.open {
		left: 0
	}
</style>