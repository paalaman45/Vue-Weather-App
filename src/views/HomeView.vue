<template>
  <main class="container text-white"> 
    <div class="pt-4 mb-8 relative">
      <input 
        type="text" 
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]" 
        placeholder="Search for a city or state"
        v-model="searchQuery"
        @input="getSearchResults"
      />

      <ul 
        v-if="mapboxSearchResults"
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]">
        
        <p v-if="searchError">Sorry, something went wrong, please try again.</p>
        <p v-if="!serverError && mapboxSearchResults.length ===0">No results match your query, try a different term.</p>
        
        <template v-else>
          <li 
          v-for="searchResult in mapboxSearchResults" :key="searchResult.id"
          @click="previewCity(searchResult)"
          class="py-2 cursor-pointer">
          {{ searchResult.properties.full_address }}
        </li>
        </template>
        
      </ul>
    </div>

    <div class="flex flex-col gap-4">
      <Suspense>
        <CityList />
        <template #fallback>
          <p>Loading...</p>
        </template>
      </Suspense>
    </div>
  </main>
</template>

<script setup>
  import { ref } from 'vue';
  import axios from 'axios';
  import { useRouter } from 'vue-router';
  import CityList from '@/components/CityList.vue';

  const router = useRouter();
  const previewCity = (searchResult) => {
    console.log(searchResult);
    const [city, state] = searchResult.properties.full_address.split(",");
    router.push({
      name: 'cityView',
      params: {state: state.replaceAll(" ", ""), city: city},
      query: {
        lat: searchResult.geometry.coordinates[1],
        lng: searchResult.geometry.coordinates[0],
        preview: true
      }
    });
  };

  const mapboxAPIKey = "pk.eyJ1IjoicGFhbGFtYW40NSIsImEiOiJjbHptbXdjdXMwYmpyMnJwbmtncXc1aWVxIn0.CMyp5zAlMGIYoHSXGcRiEg";
  const searchQuery = ref("");

  // Delay typing before execute
  const queryTimeout = ref(null);
  const mapboxSearchResults = ref(null);
  const searchError = ref(null);

  // Search functions
  const getSearchResults = () => {
    clearTimeout(queryTimeout.value);
    queryTimeout.value = setTimeout(async() => {
      if(searchQuery.value !== ''){
        try{
          const result = await axios.get(`https://api.mapbox.com/search/geocode/v6/forward?q=${searchQuery.value}&access_token=${mapboxAPIKey}&types=place`);
          mapboxSearchResults.value = result.data.features;
          console.log(mapboxSearchResults.value)
          return;
        } catch {
          searchError.value = true;
        }
        
      }
      mapboxSearchResults.value = null;
    }, 300);
  };
</script>
