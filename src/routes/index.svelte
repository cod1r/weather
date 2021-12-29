<script>
	import { onMount } from 'svelte';
	let coords = null;
	let gridpointforecast = null;
	let forecastIndex = 0;

	onMount(() => {
		navigator.geolocation.getCurrentPosition((pos) => (coords = pos.coords));
	});

	$: {
		if (coords !== null) {
			fetch(`https://api.weather.gov/points/${coords.latitude},${coords.longitude}`).then(
				async (res) => {
					let pointobject = await res.json();
					gridpointforecast = (await (await fetch(pointobject['properties']['forecast'])).json())[
						'properties'
					]['periods'];
				}
			);
		}
	}

	$: if (forecastIndex < 0 && gridpointforecast !== null)
		forecastIndex = gridpointforecast.length - 1;

	$: if (gridpointforecast !== null && forecastIndex >= gridpointforecast.length) forecastIndex = 0;
</script>

<h1 class="center sans-serif">weather forecast</h1>
{#if gridpointforecast !== null}
	<div class="flex flex-col items-center">
		<h4>{gridpointforecast[forecastIndex]['name']}</h4>
		<div>{new Date(gridpointforecast[forecastIndex]['startTime']).toUTCString()}</div>
		<div>{new Date(gridpointforecast[forecastIndex]['endTime']).toUTCString()}</div>
		<div>
			Temp: {gridpointforecast[forecastIndex]['temperature']}
			{gridpointforecast[forecastIndex]['temperatureUnit']}
		</div>
		<div>Wind Speed: {gridpointforecast[forecastIndex]['windSpeed']}</div>
		<div>Wind Direction: {gridpointforecast[forecastIndex]['windDirection']}</div>
		<p style="word-wrap: break-word; max-width: 25%; text-align: center;">
			{gridpointforecast[forecastIndex]['detailedForecast']}
		</p>
		<img src={gridpointforecast[forecastIndex]['icon']} alt="icon" />
		<div style="padding: .2rem;">
			<button on:click={() => forecastIndex--} style="">left</button>
			<button on:click={() => forecastIndex++} style="">right</button>
		</div>
	</div>
{:else}
	<h1>loading...</h1>
{/if}

<style>
	.center {
		text-align: center;
	}
	.sans-serif {
		font-family: sans-serif;
	}
	.flex {
		display: flex;
	}
	.flex-col {
		flex-direction: column;
	}
	.justify-center {
		justify-content: center;
	}
	.items-center {
		align-items: center;
	}
</style>
