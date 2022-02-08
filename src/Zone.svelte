<script>
	import { tweened } from "svelte/motion";
	import { cubicInOut } from "svelte/easing";
	import { createEventDispatcher } from "svelte";
	import TimeText from "./TimeText.svelte";

	const dispatch = createEventDispatcher();

	export let timezone;
	export let name = timezone;
	export let date = new Date();
	export let showSecond = false;

	let locale;
	function timeInfo(date, timezone) {
		let timeinfo = {};
		date.setMilliseconds(0);
		timeinfo.lzone = -date.getTimezoneOffset();
		let datetime = date.toLocaleString("sv", {
			timeZone: timezone,
		});
		let fulldate = date.toLocaleString("sv", {
			timeZone: timezone,
			timeZoneName: "short",
		});
		timeinfo.date = datetime.substring(0, 10);
		if (!showSecond) {
			timeinfo.time = datetime.substring(11, 16);
		} else {
			timeinfo.time = datetime.substring(11);
		}
		timeinfo.zonename = fulldate.substring(datetime.length);
		let ms = Date.parse(datetime);
		timeinfo.zone = (ms - date) / 60000 + timeinfo.lzone;
		timeinfo.home = timeinfo.zone == timeinfo.lzone;
		timeinfo.percentOfDay = calPercentOfDay(ms, timeinfo.lzone);
		return timeinfo;
	}

	function init() {
		try {
			locale = timeInfo(date, timezone);
		} catch (e) {
			console.log(e);
		}
	}
	init();

	function calPercentOfDay(ms, lzone) {
		return (100 * Math.floor((ms / 60000 + lzone) % (24 * 60))) / (24 * 60);
	}

	const fmin = tweened(locale.percentOfDay, {
		duration: 1000,
		easing: cubicInOut,
		interpolate: (a, b) => {
			let d = b - a;
			if (Math.abs(d) < 50) {
				return (t) => {
					return a + d * t;
				};
			}
			if (d > 0) {
				a += 100;
			} else {
				b += 100;
			}
			d = b - a;
			return (t) => {
				return (a + d * t) % 100;
			};
		},
	});

	function convertToUTC(input) {
		if (input.match(/[^0-9:]/)) {
			return null;
		}
		var digits = input.split(":");
		if (digits.length > 3) {
			return null;
		}

		if (digits.length == 1) {
			// special case for 4-digit time
			if (digits[0] >= 100 && digits[0] <= 2359) {
				digits = [Math.floor(digits[0] / 100), digits[0] % 100];
			}
		}

		// validate input
		if (digits[0] > 24) {
			return null;
		}
		if (digits.length > 1 && digits[1] > 60) {
			return null;
		}
		if (digits.length > 2 && digits[2] > 60) {
			return null;
		}

		// offset to utc for changing h/m/s
		var d = new Date().valueOf();
		let zone = locale.zone * 60 * 1000;
		d += zone;
		let t = d % (24 * 60 * 60 * 1000);
		let h, m, s;
		h = digits[0];
		if (digits.length > 1) {
			m = digits[1];
		} else {
			m = Math.floor(t / (60 * 1000)) % 60;
		}
		if (digits.length > 2) {
			s = digits[2];
		} else {
			s = Math.floor(t / 1000) % 60;
		}
		d -= d % (24 * 60 * 60 * 1000);
		d += h * 60 * 60 * 1000 + m * 60 * 1000 + s * 1000;
		d -= zone;
		return new Date(d);
	}

	function handleTimeChange(e) {
		var d = convertToUTC(e.detail.time);
		if (d) {
			dispatch("timeChange", {
				time: d,
			});
		}
	}

	$: if (locale.zonename) {
		locale = timeInfo(date, timezone);
		fmin.set(locale.percentOfDay);
	}
</script>

{#if locale.zonename}
	<zone style="--gradient-pos: {$fmin}%">
		<box class:home={locale.home}>
			<name>{name}</name>
			<timezone>
				{locale.zonename}
				({locale.zone > 0 ? "+" : ""}{locale.zone / 60})
			</timezone>
			<date>{locale.date}</date>
			<TimeText time={locale.time} on:timeChange={handleTimeChange} />
		</box>
	</zone>
{/if}

<style>
	:root {
		--gradient-dir: to top;
		--box-dir: to bottom;

		--zone-dir: column;
		--box-padding: 3em 0;
	}
	@media all and (orientation: portrait) {
		:root {
			--box-dir: to right;
			--zone-dir: row;
			--box-padding: 0 3em;
		}
	}
	zone {
		color: var(--fg-color);
		position: relative;
		display: flex;
		flex-direction: var(--zone-dir);
		justify-content: center;
		min-width: 8rem;

		background-image: linear-gradient(
			var(--gradient-dir),
			rgb(0, 0, 0) calc(calc(5% - var(--gradient-pos)) * 10),
			rgb(54, 69, 164) calc(calc(25% + 5% - var(--gradient-pos)) * 10),
			rgb(255, 250, 162) calc(calc(50% + 5% - var(--gradient-pos)) * 10),
			hsl(27, 100%, 58%) calc(calc(75% + 5% - var(--gradient-pos)) * 10),
			rgb(0, 0, 0) calc(calc(100% + 5% - var(--gradient-pos)) * 10)
		);
	}
	box {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding: var(--box-padding);
		background-image: linear-gradient(
			var(--box-dir),
			rgba(0, 0, 0, 0) 0%,
			17%,
			var(--bg-color) 40% 60%,
			83%,
			rgba(0, 0, 0, 0) 100%
		);
	}
	box::before {
		content: "";

		width: 1.5rem;
		height: 1.5rem;
		margin-top: -1.5rem;
	}
	.home::before {
		-webkit-mask-image: url("/images/home.svg");
		mask-image: url("/images/home.svg");
		background-color: var(--fg-color);
	}
</style>
