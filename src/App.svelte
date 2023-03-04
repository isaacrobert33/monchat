<script>
	import ChatContainer from "./components/ChatContainer.svelte";
	import ListContainer from "./components/ListContainer.svelte";
</script>

<div class="App">
	<ListContainer />
	<ChatContainer />
</div>

<style>
	.App {
		width: 100%;
		height: 100%;
		margin: 0;
		padding: 0;
		background-color: #222e35;
	}
	:global(body) {
		margin: 0;
		padding: 0;
	}
</style>