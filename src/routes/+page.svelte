<div class="flex flex-row h-full gap-5">
<div class="h-full w-1/4">
	<!-- Number of lines in Main Memory -->
	{#each new Array(1024) as _, index}
	<div class="border h-1/16 w- flex justify-center items-center">Main Memory</div>
	{/each}
</div>
<div class="h-full w-1/4">
	<!-- Number of lines in L2 -->
	{#each new Array(256) as _, index}
	<div class="border h-1/16 w-full">L2</div>
	{/each}
</div>
<div class="h-full w-1/4">
	<!-- Number of lines in L1 -->
    {#each new Array(128) as _, index}
      <div class="border h-1/16 w-full">L1</div>
    {/each}
	<br />
	<div>
		<!-- Number of lines in Victim Cache -->
		{#each new Array(4) as _, index}
			<div class="border h-1/2 w-full">Victim Cache</div>
		{/each}
	</div>
</div>
<div class="h-1/6 w-1/4">
	<div class="h-1/6 border justify-center items-center flex">Processor</div>
</div>
</div>
