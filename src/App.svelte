<script lang="ts">
   //@ts-nocheck
   import { onMount } from "svelte";
   let ol = {};
   let targetCountry = '';
   let selectedCountries: string[] = [];  // List of selected countries

   onMount(async () => {
      let parser = new DOMParser();
      let text: any = await fetch('europe_only_worldmap.svg');
      text = await text.text();
      let xmlDoc = parser.parseFromString(text, "text/xml");
      let paths = xmlDoc.getElementsByTagName("svg")[0].children;
      for (let i = 0; i < paths.length; i++) {
         ol[paths[i].getAttribute('title')] = paths[i].getAttribute('d');
      }

      // Select a random country
      const countryNames = Object.keys(ol);
      targetCountry = countryNames[Math.floor(Math.random() * countryNames.length)];
      document.querySelector('input')!.value = targetCountry;
   });

   // Reset all countries to the default color
   function resetAllColors() {
      const paths = document.querySelectorAll('path');
      paths.forEach(path => {
         path.setAttribute('fill', 'rgb(60,60,60)'); // Default color
         path.setAttribute('stroke', 'black');
      });
   }

   // Handle country selection from the map
   function selectCountry(country: string) {
      if (!selectedCountries.includes(country)) {
         selectedCountries.push(country);  // Add to list if not present
      }

      // Check if the selected country is correct
      const isCorrect = country === targetCountry;

      // Update the path color on the map
      updatePathColor(country, isCorrect);
   }

   // Change the path color
   function updatePathColor(country: string, isCorrect: boolean) {
      const paths = document.querySelectorAll('path');
      paths.forEach(path => {
         if (path.getAttribute('title') === country) {
            path.setAttribute('fill', isCorrect ? 'lightgreen' : 'lightcoral');
            path.setAttribute('stroke', 'black');
         }
      });
   }

   // Zoom variables
   let scale = 3;  // Méret alapértelmezett értéke duplázva
   function zoomIn() {
      scale *= 1.2;
   }

   function zoomOut() {
      scale = Math.max(scale / 1.2, 2); // Prevent scaling below 2 (az új alapértelmezett méret)
   }
</script>

<main>
   <div class="map-container">
      <svg viewBox="0 0 1009.6727 665.96301" preserveAspectRatio="xMidYMid meet" transform={`scale(${scale})`}>
         {#each Object.entries(ol) as [t, d]}
            <path d={d} title={t} 
               fill="rgb(60,60,60)"
               stroke-width="0.02"
               stroke="black"
               on:click={() => selectCountry(t)}
            ></path>
         {/each}
      </svg>
   </div>

   <!-- Randomly selected country name -->
   <div>
      <select bind:value={targetCountry} on:change={() => resetAllColors()}>
         {#each Object.entries(ol).sort() as [t, d]}
            <option value={t}>{t}</option>
         {/each}
      </select>
   </div>
   
   <!-- Zoom Controls -->
   <div class="zoom-controls">
      <button on:click={zoomIn}>+</button>
      <button on:click={zoomOut}>-</button>
   </div>
</main>

<style>
   main {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
   }
   .map-container {
      display: flex;
      justify-content: center;
      align-items: center;
      width: 80vw;
      height: 60vh;
      margin-top: 100px;
      transform: translateX(-35%); /* SVG balra tolása 50%-kal */
   }
   svg {
      width: 100%;
      height: 100%;
   }
   .zoom-controls {
      position: fixed;
      bottom: 20px;
      right: 20px;
      display: flex;
      gap: 10px;
   }
   button {
      font-size: 18px;
      padding: 8px;
      cursor: pointer;
   }
   select{
      font-family: 'Times New Roman', Times, serif;
      font-size: 20px;
      position: fixed;
      top: 85px;
      left: 50px;
      width: 250px;
      padding: 10px;
      text-align: center;
      border: 1px solid #ccc;
   }
</style>
