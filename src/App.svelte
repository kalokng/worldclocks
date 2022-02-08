<script lang="ts">
	import Zone from "./Zone.svelte";
	import Button from "./Button.svelte";

	$: date = new Date();

	let showSecond = false;

	function nextUpdate(date: Date, showSecond = false) {
		let next = 0;
		if (!showSecond) {
			next = (59 - date.getSeconds()) * 1000;
		}
		return next + 1000 - date.getMilliseconds();
	}

	let showNowBtn = false;
	let timer;
	let updateLoop = () => {
		if (!showNowBtn) {
			date = new Date();
			timer = setTimeout(updateLoop, nextUpdate(date, showSecond));
		}
	};
	timer = setTimeout(updateLoop, nextUpdate(new Date(), showSecond));

	$: zones = [
		"America/Los_Angeles",
		"America/New_York",
		"utc",
		"Europe/Berlin",
		"hongkong",
		"Australia/Melbourne",
		"Australia/Darwin",
	];

	function handleClick() {
		clearTimeout(timer);
		showNowBtn = false;
		timer = updateLoop();
	}

	function handleTimeChange(e) {
		clearTimeout(timer);
		showNowBtn = true;
		date = e.detail.time;
	}
</script>

<main class="main">
	{#each zones as zone}
		<Zone
			{date}
			{showSecond}
			timezone={zone}
			on:timeChange={handleTimeChange}
		/>
	{/each}
	{#if showNowBtn}
		<div class="now-btn"><Button on:click={handleClick}>Now</Button></div>
	{/if}
</main>

<style>
	:global(body) {
		padding: 0;
	}
	:root {
		--grid-dir: column;
		--fg-color: rgb(32, 32, 32);
		--bg-color: rgba(255, 255, 255, 0.4);
	}
	@media (prefers-color-scheme: dark) {
		:root {
			--fg-color: rgb(224, 224, 224);
			--bg-color: rgba(0, 0, 0, 0.4);
		}
	}
	@media (orientation: portrait) {
		:root {
			--grid-dir: row;
		}
	}
	.main {
		position: relative;
		display: grid;
		grid-auto-flow: var(--grid-dir);
		height: 100%;
		width: 100%;
		box-sizing: border-box;
		align-content: stretch;
	}
	.now-btn {
		position: fixed;
		left: 50%;
		top: 50%;
		transform: translate(-50%, calc(50% + 5em));
	}
</style>
