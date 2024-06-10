<script lang="ts">
	import { onMount } from 'svelte';
	import Chart from 'chart.js/auto';

	let extrusionLengths = [0.0, 0.2, 0.4, 0.6, 0.8, 1.5, 2.0, 3.0, 5.0, 10.0];

	let defaultFlow = [0.0, 0.4444, 0.6145, 0.7059, 0.7619, 0.8571, 0.8889, 0.9231, 0.952, 1.0];

	let A = [...defaultFlow];
	let B = [...defaultFlow];

	let chart: Chart;
	let outputData: { Default: string; A: string; B: string } = {
		Default: '',
		A: '',
		B: ''
	};
	let canvas: HTMLCanvasElement;

	function updateOutputData() {
		outputData = {
			Default: extrusionLengths.map((length, index) => `${length},${defaultFlow[index]}`).join(';'),
			A: extrusionLengths.map((length, index) => `${length},${A[index]}`).join(';'),
			B: extrusionLengths.map((length, index) => `${length},${B[index]}`).join(';')
		};

		saveData();
	}

	function updateChartData() {
		if (chart) {
			chart.destroy();
		}

		chart = new Chart(canvas, {
			type: 'line',
			data: {
				labels: extrusionLengths,
				datasets: [
					{
						label: 'Default',
						data: defaultFlow,
						borderColor: 'blue',
						tension: 0.2
					},
					{
						label: 'A',
						data: A,
						borderColor: 'red',
						tension: 0.2
					},
					{
						label: 'B',
						data: B,
						borderColor: 'yellow',
						tension: 0.2
					}
				]
			},
			options: {
				scales: {
					x: {
						title: {
							display: true,
							text: 'Extrusion Length'
						}
					},
					y: {
						title: {
							display: true,
							text: 'Flow Rate'
						}
					}
				}
			}
		});
	}

	onMount(() => {
		loadData();
		updateOutputData();
		updateChartData();
	});

	function saveData() {
		localStorage.setItem('flowRatesA', JSON.stringify(A));
		localStorage.setItem('flowRatesB', JSON.stringify(B));
	}

	function loadData() {
		const storedA = localStorage.getItem('flowRatesA');
		const storedB = localStorage.getItem('flowRatesB');
		if (storedA) {
			A = JSON.parse(storedA);
		}
		if (storedB) {
			B = JSON.parse(storedB);
		}
	}

	function copyToClipboard(text: string) {
		navigator.clipboard.writeText(text);
	}
</script>

<main class="container mx-auto p-4">
	<h1 class="text-3xl font-bold text-center text-primary my-4">
		Small Area Flow Compensation Helper
	</h1>

	<div class="prose my-4">
		<p>
			View the original Printables page:
			<a
				href="https://www.printables.com/model/904788-small-area-flow-compensation-tester"
				target="_blank"
				rel="noopener noreferrer"
				class="link">
				https://www.printables.com/model/904788-small-area-flow-compensation-tester
			</a>
		</p>
	</div>

	<h2 class="text-2xl font-semibold text-primary my-4">Compensation Tuning</h2>

	<div class="flex flex-col md:flex-row gap-2">
		<table class="table table-md w-full md:w-1/3">
			<thead>
				<tr>
					<th>Length</th>
					<th>Default</th>
					<th>Config A</th>
					<th>Config B</th>
				</tr>
			</thead>
			<tbody>
				{#each extrusionLengths as length, i}
					<tr>
						<td>{length}</td>
						<td>{defaultFlow[i].toFixed(4)}</td>
						<td>
							<input
								type="number"
								bind:value={A[i]}
								step="0.0001"
								class="input input-bordered input-sm w-24"
								on:change={updateOutputData}
								on:change={updateChartData} />
						</td>
						<td>
							<input
								type="number"
								bind:value={B[i]}
								step="0.0001"
								class="input input-bordered input-sm w-24"
								on:change={updateOutputData}
								on:change={updateChartData} />
						</td>
					</tr>
				{/each}
			</tbody>
		</table>

		<div class="flex flex-col w-full md:w-2/3 gap-2">
			<canvas bind:this={canvas} class="w-full pt-4" />
			<iframe
				class="w-full aspect-video"
				height="480"
				width="720"
				src="https://www.youtube.com/embed/1NBtx1K98RU"
				title="How to tune Small Area Flow Compensation to improve 3D print quality"
				frameborder="0"
				allow="accelerometer; encrypted-media; gyroscope; picture-in-picture; web-share"
				referrerpolicy="strict-origin-when-cross-origin"
				allowfullscreen />
		</div>
	</div>

	<h2 class="text-2xl font-semibold text-primary my-4">
		Compensation Model Output
		<span class="text-sm text-gray-500"> - Click to copy data</span>
	</h2>

	<div class="gap-2 mt-2">
		<div>
			<button class="btn btn-md m-2" on:click={() => copyToClipboard(outputData.Default)}>
				Default: {outputData.Default}
			</button>
		</div>
		<div>
			<button class="btn btn-md m-2" on:click={() => copyToClipboard(outputData.Default)}>
				Config A: {outputData.A}
			</button>
		</div>
		<div>
			<button class="btn btn-md m-2" on:click={() => copyToClipboard(outputData.Default)}>
				Config B: {outputData.B}
			</button>
		</div>
	</div>
</main>
