<script>
	/**
	 * @type {string}
	 */
	let option;
	let memory = {}
	let caches = {}
</script>
<div class="flex flex-row h-full gap-5 p-5">
<div class="h-full w-1/4">
	<!-- Number of lines in Main Memory -->
	{#each new Array(1024) as _, index}
	<div class=" h-1/16 border rounded-lg w- flex justify-center items-center p-1">Main Memory</div>
	{/each}
</div>
<div class="h-full w-1/4">
	<!-- Number of lines in L2 -->
	{#each new Array(256) as _, index}
	<div class="border h-1/16 w-full rounded-lg p-1">L2</div>
	{/each}
</div>
<div class="h-full w-1/4 overflow-auto border">
	<!-- Number of lines in L1 -->
    {#each new Array(128) as _, index}
    <div class="border h-1/16 w-fit rounded-lg p-1 flex flex-row">
		<div>L1</div>
		<!-- {#each new Array(64) as _, index}
		<div class="">0</div>
		{/each} -->
	</div>
    {/each}
	<br />
	<div>
		<!-- Number of lines in Victim Cache -->
		{#each new Array(4) as _, index}
			<div class="border h-1/2 w-full rounded-lg p-1">Victim Cache</div>
		{/each}
	</div>
</div>
<div class="w-1/4">
	<div class="h-96 border rounded-lg">
		<div class="mt-16 flex justify-center">
			<select bind:value={option} class="flex mr-5 text-white p-1 rounded-lg bg-blue-500 bg-opacity-30">
				<option selected value="">Select Option</option>
				<option value="load">Load</option>
				<option value="store">Store</option>
			</select>
			<input type="text" class="bg-white bg-opacity-20 rounded-lg">
		</div>
		<div class="pt-10 flex justify-center">
			{#if option == "load"}
			<button class="border rounded-lg p-1 bg-blue-800 bg-opacity-20">Load</button>
			{:else if option == "store"}
			<button class="border rounded-lg p-1 bg-blue-800 bg-opacity-20">Store</button>
			{:else if option == ""}
			<button class="border rounded-lg p-1 bg-blue-800 bg-opacity-20" disabled>Choose option first</button>
			{/if}
			</div>
		<div class="h-10 mt-10 px-2 flex">
			<div class="border rounded-l-lg h-full w-1/3"></div>
			<div class="border h-full w-1/3"></div>
			<div class="border rounded-r-lg h-full w-1/3"></div>
		</div>
		<div class="px-2 mt-2">
			<div class="border rounded-lg px-2 flex justify-center pb-20">Information</div>
		</div>
	</div>
</div>
</div>
