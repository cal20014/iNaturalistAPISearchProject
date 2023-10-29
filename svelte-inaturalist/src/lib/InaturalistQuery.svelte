<!-- Javascript -->
<script>
  export let baseUrl = '';
  let searchQuery = '';
  let observationDate = ''; // Variable for observation date
  let placeResults = [];
  let identificationResults = [];
  let selectedPlaceId = null;
  let searchPerformed = false; 

  async function searchPlaces() {
    const url = `${baseUrl}places/autocomplete?q=${searchQuery}`;
    const response = await fetch(url);
    const apiData = await response.json();
    placeResults = apiData.results || [];
    searchPerformed = true;
  }

  async function searchIdentifications() {
    if (!selectedPlaceId) return;

    const url = `${baseUrl}identifications?place_id=${selectedPlaceId}`;
    const response = await fetch(url);
    const apiData = await response.json();
    
    let rawResults = apiData.results || [];
    let uniqueTaxonNames = new Set();
    
    identificationResults = rawResults.filter(identification => {
      let taxonName = identification.observation.taxon.name;
      let observationOn = identification.observation.observed_on;
      
      let isUnique;
      if (uniqueTaxonNames.has(taxonName)) {
        isUnique = false;
      } else {
        isUnique = true;
      }

      let isWithinDate;
      if (observationDate) {
        isWithinDate = (observationOn === observationDate);
      } else {
        isWithinDate = true;
      }


      if (isUnique && isWithinDate) {
        uniqueTaxonNames.add(taxonName);
        return true;
      }
      return false;
    });
  }

  function selectPlace(id) {
    selectedPlaceId = id;
    searchIdentifications();
  }
</script>

<!-- CSS -->
<style>
  .inaturalist-container {
    font-family: Arial, sans-serif;
    max-width: 800px;
    margin: auto;
    padding: 20px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    border-radius: 8px;
    background-color: #f9f9f9;
  }

  .title {
    font-size: 2.5rem;
    color: #333;
    text-align: center;
  }

  .search-form {
    margin-bottom: 20px;
  }

  .search-input {
    width: calc(100% - 120px);
    padding: 10px;
    margin-right: 10px;
    border: 1px solid #ccc;
    border-radius: 4px;
  }

  .search-button {
    width: 20%;
    padding: .5rem;
    margin: 1rem;
    border: none;
    background-color: #005740;
    color: white;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.3s;
  }

  .search-button:hover {
    background-color: #098161;
  }

  .place-results, .identification-results {
    margin-top: 20px;
  }

  .place-results div, .identification-results li {
    background-color: white;
    margin-bottom: 10px;
    padding: 15px;
    border-radius: 4px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  }



  .place-results button {
    background-color: #f0ad4e;
    width: auto;
    padding: 5px 10px;
    font-size: 0.9em;
  }

  .place-results button:hover {
    color: white;
    border: 1px solid #f0ad4e;
    transition: background-color 0.3s;
  }

  .identification-list {
    list-style: none;
    padding: 0;
  }
</style>

<!-- HTML -->
<div class="inaturalist-container">
  <h1 class="title">iNaturalist API Search</h1>
  <form class="search-form" on:submit|preventDefault={searchPlaces}>
    <input class="search-input" type="text" bind:value={searchQuery} placeholder="Search for places">
    {#if searchPerformed}
      <input class="search-input" type="date" bind:value={observationDate} placeholder="Observation Date">
    {/if}
    
    <button class="search-button" type="submit">Search</button>
  </form>

  <div class="place-results">
    {#each placeResults as place}
      <div>
        <button on:click={() => selectPlace(place.id)}>{place.name}</button>
      </div>
    {/each}
  </div>

  <div class="identification-results">
    {#if searchPerformed}
      <h2 class="Identification-title">Identification Results</h2>
      <ul class="identification-list">
        {#each identificationResults as identification}
          <li class="identification">
            <h3 class="identification-title-name">{identification.observation.taxon.name}</h3>
            <img class="identification-img" src="{identification.observation.taxon.default_photo.square_url}" alt="{identification.observation.taxon.name}">
          </li>
        {/each}
      </ul>
    {/if}
  </div>
</div>