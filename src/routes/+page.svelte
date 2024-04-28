<script>
	let address = '';

	const memory_lines = Math.pow(2, 10);
	const l1_cache_lines = 128;
	const l1_victim_cache_lines = 4;
	const l2_cache_lines = 256;

	// Creating a memory array filled with random integers
	const memory = Array.from({ length: memory_lines }, () =>
		Array.from({ length: 64 }, () => Math.floor(Math.random() * 9) + 1)
	);
	let l2_temp = Array.from({ length: l2_cache_lines }, () => Array(64).fill(0));;
	// Initializing cache arrays with zeros
	let l2 = Array.from({ length: l2_cache_lines }, () => Array(64).fill(0));
	let l1 = Array.from({ length: l1_cache_lines }, () => Array(64).fill(0));
	let victim_cache = Array.from({ length: l1_victim_cache_lines }, () => Array(64).fill(0));
	// Tracking access history for LRU replacement in victim cache
	const lru_tracker = [];

	// Initializing counters for cache hits and misses
	let l1_hits = 0;
	let l1_misses = 0;
	let l2_hits = 0;
	let l2_misses = 0;
	let victim_cache_hits = 0;
	let victim_cache_misses = 0;

	// Initialize address list for victim cache
	let adr = [];

	// Function to extract byte offset for L1 cache
	function l1_byte_offset(address) {
		return parseInt(address.slice(-6), 2);
	}

	// Function to extract byte offset for L2 cache
	function l2_byte_offset(address) {
		return parseInt(address.slice(-6), 2);
	}

	// Function to write data to L1 cache
	function l1_write(address) {
		const line = parseInt(address, 2) >> 6;
		l1[line % 128] = [...memory[line]];
	}

	// Function to read data from L1 cache
	function l1_read(address) {
		const line = parseInt(address, 2) >> 6;
		return l1[line % 128][l1_byte_offset(address)];
	}

	// Function to check if data is in L2 cache
	function l2_check(address) {
		const line = (parseInt(address, 2) >> 6) % 256;
		const data = memory[parseInt(address, 2) >> 6];
		for (let i = 0; i < 4; i++) {
			if (JSON.stringify(l2[line + i]) === JSON.stringify(data)) {
				return true;
			}
		}
		return false;
	}

	// Function to read data from L2 cache
	function l2_read(address) {
		const line = (parseInt(address, 2) >> 6) % 256;
		for (let i = 0; i < 4; i++) {
			if (JSON.stringify(l2[line + i]) === JSON.stringify(memory[parseInt(address, 2) >> 6])) {
				return l2[line + i][l2_byte_offset(address)];
			}
		}
	}

	// Function to write data to L2 cache
	function l2_write(address) {
		const line = (parseInt(address, 2) >> 6) % 256;
		const data = memory[parseInt(address, 2) >> 6];
		let changed = 0;
		for (let i = 0; i < 4; i++) {
			if (JSON.stringify(l2[line % 256]) !== JSON.stringify(Array(64).fill(0))) {
				l2[line % 256] = [...memory[line]];
				changed = 1;
			}
		}
		if (!changed) {
			l2.splice(line, 1, ...Array(1).fill([...data]));
		}
		l2_temp = l2;
	}
	// Function to read data from victim cache
	function victim_cache_read(address, adr) {
		const index = adr.indexOf(parseInt(address, 2));
		return victim_cache[index][l1_byte_offset(parseInt(address, 2))];
	}

	// Function to write data to victim cache
	function victim_cache_write(address, adr) {
		const line = parseInt(address, 2) >> 6;
		const index = adr.indexOf(line);
		if (index !== -1) {
			lru_tracker.splice(lru_tracker.indexOf(line), 1);
			lru_tracker.push(line);
			return;
		}
		if (adr.length < l1_victim_cache_lines) {
			adr.push(line);
			victim_cache[adr.indexOf(line)] = [...memory[line]];
			lru_tracker.push(line);
		} else {
			const oldest = lru_tracker.shift();
			victim_cache[adr.indexOf(oldest)] = [...memory[line]];
			adr.splice(adr.indexOf(oldest), 1);
			adr.push(line);
			lru_tracker.push(line);
		}
	}

	// Function to check if data is in victim cache
	function victim_cache_check(address, adr) {
		return adr.includes(parseInt(address, 2));
	}

	// Function to check if data is in L1 cache
	function l1_check(address) {
		const line = parseInt(address, 2) >> 6;
		const data = memory[line];
		return JSON.stringify(l1[line % 128]) === JSON.stringify(data);
	}

	// Function to simulate processor operations
	function manualInput() {
	
		if (address.length === 16 && /^[01]+$/.test(address)) {
			processAddress(address);
			console.log(l2[0]);
		} else {
			alert('Invalid address. Please enter a 16-bit binary number.');
		}
	}

	// Function to process address and simulate cache operations
	function processAddress(address) {
		console.log('ADDRESS ISSUED BY PROCESSOR:', address, ' ');

		// Check if data is in L1 cache
		const l1_cache_hit = l1_check(address);
		if (l1_cache_hit) {
			console.log('L1 CACHE HIT', ' ');
			const data = l1_read(address);
			console.log('DATA:', data, ' ');
			l1_hits++;
		} else {
			console.log('L1 CACHE MISS', ' ');
			l1_misses++;
			l1_write(address); // If miss, write data to L1 cache


			// Check if data is in victim cache
			const vcache_hit = victim_cache_check(address, adr);
			if (vcache_hit) {
				console.log('VICTIM CACHE HIT', ' ');
				const data = victim_cache_read(address, adr);
				console.log('DATA:', data, ' ');
				victim_cache_hits++;
			} else {
				console.log('VICTIM CACHE MISS', ' ');
				victim_cache_misses++;
				victim_cache_write(address, adr); // If miss, write data to victim cache

				// Check if data is in L2 cache
				const l2_cache_hit = l2_check(address);
				if (l2_cache_hit) {
					console.log('L2 CACHE HIT', ' ');
					const data = l2_read(address);
					console.log('DATA:', data, ' ');
					l2_hits++;
				} else {
					console.log('L2 CACHE MISS');
					l2_misses++;
					l2_write(address); // If miss, write data to L2 cache
				}
			}
		}
		console.log();
		console.log();



	}
</script>

<div class="flex flex-row h-full gap-5 p-5">
	<div class="h-full w-1/4">
	<!-- Number of lines in Main Memory -->
	{#each memory as line, index}
		<div class=" h-1/16 border rounded-lg w- flex justify-center items-center p-1 overflow-auto">{line}</div>
	{/each}
	</div>
	<div class="h-full w-1/4">
		{#each l2_temp as line, index}
			<div class="border h-1/16 w-full rounded-lg p-1 overflow-auto">
				{line}
			</div>
		{/each}
	</div>
	<div class="h-full w-1/4 overflow-auto">
		<!-- Number of lines in L1 -->
		<!-- {#each new Array(128) as _, index}
			<div class="border h-1/16 w-full rounded-lg p-1 flex flex-row">
				L1
				<!-- {#each new Array(64) as _, index}
		<div class="">0</div>
		{/each} -->
			<!-- </div> -->
		<!-- {/each} -->
		<div>
		{#each l1 as line, index}
			<div class="border h-1/16 w-full rounded-lg p-1 overflow-auto">
					{line}
			</div>
		{/each}
		</div>
		<br />
		<div>
			<!-- Number of lines in Victim Cache -->
			<!-- {#each new Array(4) as _, index}
				<div class="border h-1/2 w-full rounded-lg p-1">Victim Cache</div>
			{/each} -->
			{#each victim_cache as line, index}
				<div class="border h-1/2 w-full rounded-lg p-1 overflow-auto">
					{line}
				</div>
			{/each}
		</div>
	</div>
	<div class="w-1/4">
		<div class="h-96 border rounded-lg">
			<div class="mt-16 flex justify-center">
				<div class="flex mr-5">Store</div>
				<input type="text" class="bg-white bg-opacity-20 rounded-lg" bind:value={address} maxlength="16" />
			</div>
			<div class="pt-10 flex justify-center">
				<button class="border rounded-lg p-1 bg-blue-800 bg-opacity-20" on:click={manualInput}>Write</button>
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
