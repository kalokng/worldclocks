<script>
	import { createEventDispatcher } from "svelte";

	const dispatch = createEventDispatcher();

	export let time;

	function handleChange(e) {
		document.activeElement.blur();
		dispatch("timeChange", { time: e.target.value });
	}

	function handleFocus(e) {
		e.target.value = time;
		e.target.select();
	}
</script>

<timetext>
	<label for="field">{time}</label>
	<input
		type="text"
		id="field"
		on:change={handleChange}
		on:focus={handleFocus}
	/>
</timetext>

<style>
	timetext {
		align-items: center;
		margin-top: 0.2em;
		padding: 0.2em 0.4em;
		font-family: monospace;
		font-weight: bold;
		font-size: 1.5em;
		border-radius: 0.3em;
		border: solid;
		border-width: 0.1em;
		height: 1em;
		position: relative;
		overflow: hidden;
	}
	input {
		position: absolute;
		top: 0;
		left: 0;
		background-color: transparent;
		color: transparent;
		border: 0;
		width: 100%;
		height: 100%;
		font-weight: bold;
	}
	input:focus {
		color: var(--fg-color);
		background-color: var(--bg-color);
	}
	input::placeholder {
		color: transparent;
	}
	label {
		user-select: none;
	}
	timetext:focus {
		color: red;
	}
</style>
