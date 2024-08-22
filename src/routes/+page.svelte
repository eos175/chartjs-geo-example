<script lang="ts">
	import { onMount } from 'svelte';
	import Chart from 'chart.js/auto';
	import type { TooltipItem, ChartConfiguration } from 'chart.js/auto';
	import {
		topojson,
		ChoroplethController,
		GeoFeature,
		ColorScale,
		ProjectionScale,
		SizeScale
	} from 'chartjs-chart-geo';

	Chart.register(ChoroplethController, GeoFeature, ColorScale, ProjectionScale, SizeScale);

	interface TooltipRawDataChoropleth {
		feature: {
			type: string;
			id: string;
			properties: {
				name: string;
			};
			geometry: {
				type: string;
				coordinates: any[];
			};
		};
		value: any;
	}

	function formatNumber(n: number) {
		if (Number.isInteger(n)) {
			return n.toString();
		}
		return n.toFixed(2);
	}

	let chartCanvas: HTMLCanvasElement;

	onMount(async () => {
		//const response = await fetch('/countries-50m.json');
		const response = await fetch('https://unpkg.com/world-atlas@2.0.2/countries-50m.json');
		const worldAtlas = await response.json();

		const countries = (topojson.feature(worldAtlas, worldAtlas.objects.countries) as any).features;

		//console.log(countries.map((d: any) => d.properties.name));

		const config: ChartConfiguration<'choropleth'> = {
			type: 'choropleth',
			data: {
				labels: countries.map((d: any) => d.properties.name),
				datasets: [
					{
						label: 'Countries',
						data: countries.map((d: any) => {
							let value = Math.random() * 100; // Personaliza los valores según tus necesidades
							if (d.properties.name === 'Brazil') {
								value = 100;
							}
							return {
								feature: d,
								value: value
							};
						})
					}
				]
			},
			options: {
				showOutline: false,
				showGraticule: false,
				plugins: {
					legend: {
						display: false
					},
					tooltip: {
						/*
						callbacks: {
							label: function (tooltipItem: TooltipItem<'choropleth'>) {
								const raw = tooltipItem.raw as TooltipRawDataChoropleth;

								//console.log(raw.feature.properties.name)

								return `${raw.feature.properties.name}: ${formatNumber(raw.value as number)}`;
							}
						}*/
					}
				},
				scales: {
					projection: {
						axis: 'x',
						projection: 'equalEarth',
						projectionScale: 1.1,
						projectionOffset: [0, 0]
					}
				}
			}
		};

		const ctx = chartCanvas.getContext('2d');
		if (ctx !== null) {
			new Chart(ctx, config);
		}
	});
</script>

<div class="flex h-screen flex-col items-center justify-center p-4">
	<h1 class="mb-3 text-3xl text-white">Chart.js with Svelte, TypeScript, and Tailwind CSS</h1>
	<div class="h-full w-full rounded-lg bg-gray-800 shadow-md">
		<canvas bind:this={chartCanvas} class="max-h-full"></canvas>
	</div>
</div>

<style>
</style>
