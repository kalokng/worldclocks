<div on:click><slot /></div>

<style>
	div {
		border: solid;
		border-radius: 3em;
		padding: 0.3em 1em;
		font-weight: bold;
		background-color: rgba(255, 255, 255, 0.7);

		transition-property: all;
		transition-duration: 400ms;
	}
	div:hover {
		box-shadow: 1px;
		filter: drop-shadow(0 0 0.3em white);
		cursor: pointer;
	}
</style>
