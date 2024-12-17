<script>
	import { onMount, setContext } from "svelte";
	import csvUrl from "./assets/points.csv";
	import Canvas from "./lib/Canvas.svelte";

	async function loadCSV(path) {
		try {
			const response = await fetch(path);
			const data = await response.text();

			const rows = data.split("\n");

			let dataObject = {};
			dataObject.metadata = [];
			if (rows.length > 0) {
				dataObject.metadata = rows[0]
					.split(",")
					.slice(0, -1)
					.map((value) => [null, null]);
			}

			dataObject.pointData = rows.map((row) => {
				const values = row.split(",").map((value) => value.trim());
				let point = [];
				values.forEach((value, index) => {
					if (index != 4) {
						value = parseFloat(value);
						if (
							dataObject.metadata[index][0] == null ||
							value < dataObject.metadata[index][0]
						) {
							dataObject.metadata[index][0] = value;
						} else if (
							dataObject.metadata[index][1] == null ||
							value > dataObject.metadata[index][1]
						) {
							dataObject.metadata[index][1] = value;
						}
					}
				});
				return [
					parseFloat(values[0]),
					parseFloat(values[1]),
					parseFloat(values[2]),
					parseFloat(values[3]),
					parseInt(values[4], 10),
				];
			});

			return dataObject;
		} catch (error) {
			console.error("Error loading CSV:", error);
			return [];
		}
	}

	let dataObject = $state(null);
	let points = $derived(dataObject?.pointData);
	let metadata = $derived(dataObject?.metadata);
	onMount(async () => {
		dataObject = await loadCSV(csvUrl);
	});
</script>

<main>
	<Canvas pointData={points} {metadata}></Canvas>
</main>

<style>
	main {
		width: 100%;
		height: 100%;
	}
</style>
