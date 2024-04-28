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
    
    // Initializing cache arrays with zeros
    let l1 = Array.from({ length: l1_cache_lines }, () => Array(64).fill(0));
    let l2 = Array.from({ length: l2_cache_lines }, () => Array(64).fill(0));
    let victim_cache = Array.from(
      { length: l1_victim_cache_lines },
      () => Array(64).fill(0)
    );
    
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
        if (JSON.stringify(l2[line + i]) !== JSON.stringify(Array(64).fill(0))) {
          l2[line + i] = [...data];
          changed = 1;
        }
      }
      if (!changed) {
        l2.splice(line, 1, ...Array(1).fill([...data]));
      }
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
      } else {
        alert('Invalid address. Please enter a 16-bit binary number.');
      }
    }
    
    // Function to process address and simulate cache operations
    function processAddress(address) {
      console.log("ADDRESS ISSUED BY PROCESSOR:", address, ' ');
    
      // Check if data is in L1 cache
      const l1_cache_hit = l1_check(address);
      if (l1_cache_hit) {
        console.log("L1 CACHE HIT", ' ');
        const data = l1_read(address);
        console.log("DATA:", data, ' ');
        l1_hits++;
      } else {
        console.log("L1 CACHE MISS", ' ');
        l1_misses++;
        l1_write(address); // If miss, write data to L1 cache
    
        // Check if data is in victim cache
        const vcache_hit = victim_cache_check(address, adr);
        if (vcache_hit) {
          console.log("VICTIM CACHE HIT", ' ');
          const data = victim_cache_read(address, adr);
          console.log("DATA:", data, ' ');
          victim_cache_hits++;
        } else {
          console.log("VICTIM CACHE MISS", ' ');
          victim_cache_misses++;
          victim_cache_write(address, adr); // If miss, write data to victim cache
    
          // Check if data is in L2 cache
          const l2_cache_hit = l2_check(address);
          if (l2_cache_hit) {
            console.log("L2 CACHE HIT", ' ');
            const data = l2_read(address);
            console.log("DATA:", data, ' ');
            l2_hits++;
          } else {
            console.log("L2 CACHE MISS");
            l2_misses++;
            l2_write(address); // If miss, write data to L2 cache
          }
        }
      }
      console.log();
      updateCacheDisplay(); // Update cache display after each iteration
      console.log();
    }
    
    // Function to update cache display
    function updateCacheDisplay() {
      const l1CacheDisplay = document.getElementById('l1-cache');
      const victimCacheDisplay = document.getElementById('victim-cache');
      const l2CacheDisplay = document.getElementById('l2-cache');
    
      // // Update cache display content based on cache arrays
      l1CacheDisplay.innerHTML = "<h3>L1 Cache:</h3>";
      for (let i = 0; i < l1_cache_lines; i++) {
        l1CacheDisplay.innerHTML += `<p>Line ${i}: ${JSON.stringify(l1[i])}</p>`;
      }
      victimCacheDisplay.innerHTML = "<h3>Victim Cache:</h3>";
      for (let i = 0; i < l1_victim_cache_lines; i++) {
        victimCacheDisplay.innerHTML += `<p>Line ${i}: ${JSON.stringify(victim_cache[i])}</p>`;
      }
      l2CacheDisplay.innerHTML = "<h3>L2 Cache:</h3>";
      for (let i = 0; i < l2_cache_lines; i++) {
        l2CacheDisplay.innerHTML += `<p>Line ${i}: ${JSON.stringify(l2[i])}</p>`;
      }
    }
  </script>
<div class="">
  <h1 class="flex justify-center text-2xl">Cache Memory Simulation</h1>
  <div class="flex justify-center">
  <div id="input-container" class="h-full w-fit flex justify-center flex-col">
    <label for="address" class="justify-center flex">Enter address in binary format (16 bits)</label>
    <input type="text" class="bg-blue-300 rounded-lg bg-opacity-25 flex justify-center" bind:value={address} maxlength="16">
    <button on:click={manualInput} class="flex justify-center">Enter</button>
  </div>
  </div>
  <div id="cache-display" class="w-full">
    <h2 class="flex justify-center text-3xl">Cache Contents</h2>
    <div id="l1-cache"></div>
    <div id="victim-cache"></div>
    <div id="l2-cache"></div>
  </div>
</div>