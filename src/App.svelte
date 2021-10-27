<script>
	import { onMount } from 'svelte'
	import { MDBCol } from "mdbsvelte";
	import Line from "svelte-chartjs/src/Line.svelte"
	import moment from 'moment'

	let dataLines = []

	async function addGraph() {
		let coin = JSON.parse(document.getElementById('coins').value),
		currency = document.getElementById('currencies').value,
		date = document.getElementById('date').value,
		coins = await fetch(`https://api.coingecko.com/api/v3/coins/${coin.id}/market_chart?vs_currency=${currency}&days=${date}`).then(x => x.json()),
		labels = [],
		data = [];

		const unique = (value, index, self) => {
			return self.indexOf(value) === index
		}

		for (let i = 0; i < coins.prices.length; i++) {
			data.push(coins.prices[i][1]);
			labels.push(moment(coins.prices[i][0]).format("D ddd"));
		}

		let dataLine = {
			labels: labels.filter(unique),
			datasets: [
				{
					label: `${coin.name} - ${currency} - ${moment(coins.prices[0][0]).format("D ddd")} to now`,
					fill: true,
					lineTension: 0.3,
					backgroundColor: "rgba(45, 83, 218, .3)",
					borderColor: "rgb(45, 83, 218)",
					borderCapStyle: "butt",
					borderDash: [],
					borderDashOffset: 0.0,
					borderJoinStyle: "miter",
					pointBorderColor: "rgb(205, 130,1 58)",
					pointBackgroundColor: "rgb(255, 255, 255)",
					pointBorderWidth: 10,
					pointHoverRadius: 5,
					pointHoverBackgroundColor: "rgb(0, 0, 0)",
					pointHoverBorderColor: "rgba(220, 220, 220,1)",
					pointHoverBorderWidth: 2,
					pointRadius: 1,
					pointHitRadius: 10,
					data: data
				}
			]
		};

		dataLines = [...dataLines, dataLine]
	}

	let coins = [], currencies = []
	onMount(async () => {
		currencies = await fetch('https://api.coingecko.com/api/v3/simple/supported_vs_currencies').then(x => x.json())
		coins = await fetch('https://api.coingecko.com/api/v3/coins/markets?vs_currency=eur&order=market_cap_desc&per_page=20&page=1&sparkline=false').then(x => x.json())
	})
</script>

<main class="max-w-screen min-h-screen w-full p-10">
	<div id="container-params" class="inline-flex items-center rounded-xl shadow-sm fixed top-10 left-10">
		<div class="flex flex-col pl-4 py-4">
			<label class="mb-3 text-white font-bold" for="coins">Coins</label>
			<select class="bg-white rounded-md shadow-sm p-4 cursor-pointer" name="coins" id="coins">
				{#each coins as coin, i}
					<option value="{JSON.stringify(coin)}">{coin.name}</option>
				{/each}
			</select>
		</div>
		<div class="flex flex-col ml-5 p-4">
			<label class="mb-3 text-white font-bold" for="currencies">Currencies</label>
			<select class="bg-white rounded-md shadow-sm p-4 cursor-pointer" name="currencies" id="currencies">
				{#each currencies as currency, i}
					<option value="{currency}">{currency}</option>
				{/each}
			</select>
		</div>
		<div class="flex flex-col ml-5 py-4 pr-4">
			<label class="mb-3 text-white font-bold" for="date">Date</label>
			<select class="bg-white rounded-md shadow-sm p-4 cursor-pointer" name="date" id="date">
				<option selected value="1">1 day</option>
				<option value="5">5 days</option>
				<option value="30">30 days</option>
				<option value="180">6 months</option>
				<option value="360">1 year</option>
				<option value="1800">5 years</option>
			</select>
		</div>
		<div on:click={addGraph} id="add-graph" class="button cursor-pointer text-white flex transition-all duration-100 ease-in-out items-center h-full w-16 justify-center rounded-tr-xl rounded-br-xl" style="height: 113px;">
			<h1 class="font-bold text-4xl">+</h1>
		</div>
	</div>
	<section id="container-graph" class="grid grid-cols-2 w-full gap-10 mt-40">
		{#if dataLines.length > 0}
			{#each dataLines as dataLine, i}
				<MDBCol class="bg-white rounded-xl shadow-sm p-4">
					<Line data={dataLine} options={{ responsive: true }}/>
				</MDBCol>
			{/each}
		{/if}
	</section>
</main>
