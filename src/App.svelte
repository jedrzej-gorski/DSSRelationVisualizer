<script>
	import { onMount, setContext } from "svelte";
	import csvUrl from "./assets/points.csv";
	import Canvas from "./lib/Canvas.svelte";

	async function loadCSV(path) {
		try {
			const response = await fetch(path);
			const data = await response.text();

			const rows = data.split("\n");

			const result = rows.map((row) => {
				const values = row.split(",").map((value) => value.trim());
				return [
					parseFloat(values[0]),
					parseFloat(values[1]),
					parseFloat(values[2]),
					parseFloat(values[3]),
					parseInt(values[4], 10),
				];
			});
			
			return result;
		} catch (error) {
			console.error("Error loading CSV:", error);
			return [];
		}
	}

	let points = []

	onMount(async () => {
		points = await loadCSV(csvUrl);
	})
</script>

<main>
	<Canvas pointData={points}></Canvas>
</main>

<style>
	main {
		width: 100%;
		height: 100%;
	}
</style>
