<script lang="ts">
  /* global google */
  import * as GMAPILoader from '@googlemaps/js-api-loader';
  const { Loader } = GMAPILoader;
  import { onMount } from 'svelte';
  import SearchBar from './components/SearchBar.svelte';
  import Sections from './sections/Sections.svelte';

  const googleMapsApiKey = import.meta.env.VITE_GOOGLE_MAPS_API_KEY;
  const defaultPlace = {
    name: 'Rinconada Library',
    address: '1213 Newell Rd, Palo Alto, CA 94303',
  };

  let location: google.maps.LatLng | undefined;
  let isSidebarOpen = false; // Track sidebar state for mobile
  const zoom = 19;

  // Initialize app.
  let mapElement: HTMLElement;
  let map: google.maps.Map;
  let geometryLibrary: google.maps.GeometryLibrary;
  let mapsLibrary: google.maps.MapsLibrary;
  let placesLibrary: google.maps.PlacesLibrary;

  // Toggle sidebar function for mobile
  const toggleSidebar = () => {
    isSidebarOpen = !isSidebarOpen;
  };

  // Popup visibility states
  let showPopup1 = true; // First popup
  let showPopup2 = false; // Second popup

  onMount(async () => {
    // Load the Google Maps libraries.
    const loader = new Loader({ apiKey: googleMapsApiKey });
    const libraries = {
      geometry: loader.importLibrary('geometry'),
      maps: loader.importLibrary('maps'),
      places: loader.importLibrary('places'),
    };
    geometryLibrary = await libraries.geometry;
    mapsLibrary = await libraries.maps;
    placesLibrary = await libraries.places;

    // Get the address information for the default location.
    const geocoder = new google.maps.Geocoder();
    const geocoderResponse = await geocoder.geocode({
      address: defaultPlace.address,
    });
    const geocoderResult = geocoderResponse.results[0];

    // Initialize the map at the desired location.
    location = geocoderResult.geometry.location;
    map = new mapsLibrary.Map(mapElement, {
      center: location,
      zoom: zoom,
      tilt: 0,
      mapTypeId: 'satellite',
      mapTypeControl: false,
      fullscreenControl: false,
      rotateControl: false,
      streetViewControl: false,
      zoomControl: false,
    });

    // Trigger resize when map is loaded to ensure proper rendering
    window.addEventListener('resize', () => {
      if (map) {
        google.maps.event.trigger(map, 'resize');
      }
    });
  });

  // Close popup 1 and show popup 2
  const closePopup1 = () => {
    showPopup1 = false;
    showPopup2 = true;
  };

  // Close popup 2
  const closePopup2 = () => {
    showPopup2 = false;
  };
</script>

<!-- Header -->
<header class="bg-[#005f7f] text-white shadow-md p-4 fixed w-full top-0 left-0 z-50">
  <div class="flex justify-between items-center">
    <h1 class="text-2xl font-semibold">Promise Energy</h1>
    <nav>
      <ul class="flex space-x-6">
        <li><a href="https://promiseenergy.com/" class="hover:underline transition-all">Home</a></li>
        <li><a href="https://promiseenergy.com/about" class="hover:underline transition-all">About</a></li>
        <li><a href="https://promiseenergy.com/contact" class="hover:underline transition-all">Contact</a></li>
      </ul>
    </nav>
  </div>
</header>

<!-- Mobile toggle button -->
<button
  class="fixed z-50 top-20 left-4 bg-white p-2 rounded-full shadow-lg md:hidden"
  on:click={toggleSidebar}
  aria-label="Toggle sidebar"
>
  <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
    {#if isSidebarOpen}
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
    {:else}
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16" />
    {/if}
  </svg>
</button>

<!-- First Popup Modal -->
{#if showPopup1}
  <div class="fixed inset-0 bg-black bg-opacity-50 flex justify-center items-center z-50">
    <div class="bg-white p-6 rounded-lg w-11/12 md:w-96">
      <h2 class="text-xl font-semibold mb-4">Understand Your Energy Costs & Save with Solar</h2>
      <p class="mb-4">This tool helps you understand your current energy costs by analyzing your consumption patterns. By assessing your usage, you can see how implementing solar solutions can significantly reduce your electricity bills and lower your environmental impact.</p>
      <p>Promise Energy offers tailored solar solutions to help you make the switch to clean, affordable energy. Start saving today and cut your costs with our personalized solar plans!</p>
      <button class="mt-4 w-full bg-[#005f7f] text-white p-2 rounded-md hover:bg-[#004a5c]" on:click={closePopup1}>Close</button>
    </div>
  </div>
{/if}

<!-- Second Popup Modal -->
{#if showPopup2}
  <div class="fixed inset-0 bg-black bg-opacity-50 flex justify-center items-center z-50">
    <div class="bg-white p-6 rounded-lg w-11/12 md:w-96">
      <h2 class="text-xl font-semibold mb-4">How to Use This Tool</h2>
      <p class="mb-4">To use this tool, simply enter your address in the search bar to get an overview of your energy consumption. You can explore how much energy your location uses and how solar energy can help lower your costs.</p>
      <p>Once you've reviewed the details, Promise Energy offers personalized plans and consultations to help you implement solar solutions that fit your needs and budget.</p>
      <button class="mt-4 w-full bg-[#005f7f] text-white p-2 rounded-md hover:bg-[#004a5c]" on:click={closePopup2}>Close</button>
    </div>
  </div>
{/if}

<div class="flex flex-col md:flex-row h-full relative mt-16"> <!-- Added margin-top to offset the fixed header -->
  <!-- Main map -->
  <div bind:this={mapElement} class="w-full h-[calc(100vh-3rem)] md:h-screen" />
  
  <!-- Side bar - responsive design -->
  <aside
    class="
      fixed md:static 
      w-full md:w-96 
      h-screen 
      bg-white
      transform transition-transform duration-300 ease-in-out
      {isSidebarOpen ? 'translate-x-0' : 'translate-x-full md:translate-x-0'}
      right-0 top-0
      z-40
      overflow-auto
      shadow-lg md:shadow-none
      {isSidebarOpen ? 'pt-20' : ''} <!-- Added padding-top when sidebar is open in mobile view -->
    "
  >
    <div class="flex flex-col space-y-2 h-full p-4">
      {#if placesLibrary && map}
        <SearchBar 
          bind:location 
          {placesLibrary} 
          {map} 
          initialValue={defaultPlace.name}
        />
      {/if}
      
      {#if location}
        <Sections 
          {location} 
          {map} 
          {geometryLibrary} 
          {googleMapsApiKey}
        />
      {/if}
      
      <div class="grow" />
    </div>
  </aside>
</div>

<!-- Footer -->
<footer class="bg-[#003c4f] text-white p-4 mt-4">
  <div class="flex justify-between items-center">
    <p>&copy; 2024 Promise Energy. All Rights Reserved.</p>
    <div>
      <a href="https://promiseenergy.com/privacy-policy" class="hover:underline">Privacy Policy</a>
      <span class="mx-2">|</span>
      <a href="https://promiseenergy.com/who-we-serve" class="hover:underline">Terms of Service</a>
    </div>
  </div>
</footer>

<style>
  /* Prevent body scroll when sidebar is open on mobile */
  :global(body.sidebar-open) {
    overflow: hidden;
  }

  /* Ensure the map container takes full height */
  :global(html, body) {
    height: 100%;
    margin: 0;
    padding: 0;
  }
</style>
