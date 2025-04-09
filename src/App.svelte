<script lang="ts">
   //@ts-nocheck
   import { onMount } from "svelte";
   let ol = {};
   let targetCountry = '';
   let selectedCountries: string[] = [];  // List of selected countries
   let gameWon = false;

   onMount(async () => {
      let parser = new DOMParser();
      let text: any = await fetch('europe_only_worldmap.svg');
      text = await text.text();
      let xmlDoc = parser.parseFromString(text, "text/xml");
      let paths = xmlDoc.getElementsByTagName("svg")[0].children;
      for (let i = 0; i < paths.length; i++) {
         ol[paths[i].getAttribute('title')] = paths[i].getAttribute('d');
      }
      selectNewTargetCountry();
   });

   function selectNewTargetCountry() {
      const countryNames = Object.keys(ol).filter(name => name && name.trim() !== '');
      targetCountry = countryNames[Math.floor(Math.random() * countryNames.length)];
      selectedCountries = [];
      gameWon = false;
      resetAllColors();
   }

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
      if (gameWon) return; // Don't allow selection after winning

      if (!selectedCountries.includes(country)) {
         selectedCountries.push(country);  // Add to list if not present
      }

      // Check if the selected country is correct
      const isCorrect = country === targetCountry;

      // Update the path color on the map
      updatePathColor(country, isCorrect);

      if (isCorrect) {
         gameWon = true;
         // Wait 2 seconds before selecting a new country
         setTimeout(selectNewTargetCountry, 2000);
      }
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
   let scale = 2.8; // Reduced scale slightly to show more of the map
   function zoomIn() {
      scale *= 1.2;
   }

   function zoomOut() {
      scale = Math.max(scale / 1.2, 0.5);
   }
</script>

<main>
   <div class="target-country">
      Find: <span class="country-name">{targetCountry}</span>
   </div>
   
   <div class="map-container">
      <div class="map-wrapper">
         <svg viewBox="0 0 1009.6727 665.96301" preserveAspectRatio="xMidYMid meet" transform="translate(-250, 30) scale({scale})">
            {#each Object.entries(ol) as [t, d]}
               <path d={d} title={t} 
                  fill="rgb(60,60,60)"
                  stroke-width="0.8"
                  stroke="black"
                  on:click={() => selectCountry(t)}
               ></path>
            {/each}
         </svg>
      </div>
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
      min-height: 100vh;
      width: 100%;
      overflow: hidden;
   }
   
   .target-country {
      color: black;
      position: fixed;
      top: 20px;
      left: 20px;
      font-size: 24px;
      font-weight: bold;
      background-color: white;
      padding: 10px 20px;
      border-radius: 5px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.2);
      z-index: 100;
      text-align: left;
   }
   
   .country-name {
      color: #d32f2f;
      text-align: left;
      display: inline-block;
      margin-left: 5px;
   }
   
   .map-container {
      display: flex;
      justify-content: center;
      align-items: center;
      width: 100%;
      height: 90vh;
      margin-top: 70px; /* Increased top margin to move map down */
   }
   
   .map-wrapper {
      position: relative;
      width: 100%;
      height: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
      overflow: visible;
   }
   
   svg {
      width: 100%;
      height: 100%;
      display: block;
   }
   
   path {
      transition: fill 0.2s ease;
      cursor: pointer;
   }
   
   path:hover {
      stroke-width: 1.5;
      stroke: #ffffff;
   }
   
   .zoom-controls {
      position: fixed;
      bottom: 20px;
      right: 20px;
      display: flex;
      gap: 10px;
      z-index: 100;
   }
   
   button {
      font-size: 18px;
      padding: 8px;
      cursor: pointer;
   }
</style>
