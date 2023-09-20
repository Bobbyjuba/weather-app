<template>
  <div v-for="city in savedCities" :key="city.id">
    <CityCard :city="city" @click="goToCityView(city)" />
  </div>

  <p v-if="savedCities.length === 0">
    No locations added. To track a location, search in the field above. 
  </p>
</template>

<script setup>
import {ref} from 'vue';
import axios from 'axios';
import CityCard from './CityCard.vue';
import { useRouter } from 'vue-router';

const savedCities = ref([])
const getCities = async () => {
  if (localStorage.getItem('savedCities')) {
    savedCities.value = JSON.parse(localStorage.getItem('savedCities')); 
  }

  const requests = [];
  savedCities.value.forEach((city) => {
    requests.push(axios.get(`https://api.openweathermap.org/data/2.5/weather?lat=${city.coords.lat}&lon=${city.coords.lng}&appid=0afef796c2d00bde5337fb3e0bfb80ed&units=imperial`))
  });

  const weatherData = await Promise.all(requests);

  // Flicker delay when loading saved city information... might remove
  await new Promise((res) => setTimeout(res, 750));

  weatherData.forEach((value, index) => {
    savedCities.value[index].weather = value.data;
  });
}

await getCities();

const router = useRouter();
const goToCityView = (city) => {
  router.push({
    name: 'cityView',
    params: {
      state: city.state,
      city: city.city,
    },
    query: {
      id: city.id,
      lat: city.coords.lat,
      lng: city.coords.lng,
    },
  });
}
</script>
