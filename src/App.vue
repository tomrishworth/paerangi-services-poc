<template>
  <div class="container py-5" style="max-width: 600px">
    <h1 class="h2">Find your nearest Vaccination Centre</h1>
    <div class="mb-4" id="geocoder"></div>
    <div class="mb-2 pb-2 border-bottom" v-for="(location, index) in locations.features" :key="index">
      <h2 class="h4 mb-1">{{ location.properties.name }}</h2>
      <div class="text-muted" v-if="location.properties.distance">
        {{ roundedDistance(location.properties.distance) }} kms away
      </div>
    </div>
  </div>
</template>

<script>
// import locationsInterestJson from './json/locations-of-interest.json';
import vaccinationCentres from './json/vaccination-centres.json';
import mapboxgl from 'mapbox-gl';
import MapboxGeocoder from '@mapbox/mapbox-gl-geocoder';
import '@mapbox/mapbox-gl-geocoder/dist/mapbox-gl-geocoder.css';
import * as turf from '@turf/turf';

export default {
  name: 'App',
  data() {
    return {
      locations: vaccinationCentres,
    };
  },
  methods: {
    roundedDistance(val) {
      return Math.round(val * 100) / 100;
    },
  },
  mounted() {
    mapboxgl.accessToken =
      'pk.eyJ1IjoidG9tcmlzaHdvcnRoIiwiYSI6IjcxZTI2YTI2Yjk2MzI1NmExY2M1ZDE1YzlkNTA3NzA2In0.v20_hLgkkO_9Y3vApGoPUA';

    const geocoder = new MapboxGeocoder({
      accessToken: mapboxgl.accessToken,
      types: 'region,place,postcode,locality,neighborhood,address',
      placeholder: 'Search for your address or suburb',
      bbox: [174.40302091538888, -37.2650250773527, 175.2091427200361, -36.554590867796996], // Set the bounding box coordinates
    });

    geocoder.addTo('#geocoder');

    // Get the geocoder results container.
    const results = document.getElementById('result');

    // Add geocoder result to container.
    geocoder.on('result', ({ result }) => {
      const searchResult = result.geometry;

      const options = { units: 'kilometers' };
      for (const location of this.locations.features) {
        let temp = JSON.stringify(location.geometry);
        let temp2 = JSON.parse(temp);

        location.properties.distance = turf.distance(searchResult, temp2, options);
      }

      this.locations.features.sort((a, b) => {
        if (a.properties.distance > b.properties.distance) {
          return 1;
        }
        if (a.properties.distance < b.properties.distance) {
          return -1;
        }
        return 0;
      });
    });

    // Clear results container when search is cleared.
    geocoder.on('clear', () => {
      results.innerText = '';
    });
  },
};
</script>

<style>
.mapboxgl-ctrl-geocoder {
  width: 100%;
  max-width: 500px;
  font-size: 18px;
}
</style>
