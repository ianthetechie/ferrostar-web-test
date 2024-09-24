<script setup lang="ts">
import {
  BrowserLocationProvider,
  FerrostarMap
} from "@stadiamaps/ferrostar-webcomponents";
import {onMounted, Ref, ref} from "vue";
import {MapLibreSearchControl} from "@stadiamaps/maplibre-search-box";
import searchBoxStyle from "@stadiamaps/maplibre-search-box/dist/style.css?inline";

// If you need very fine-grained control over the component,
// you can use the Vue `ref` to keep a reference and react to state changes
// elsewhere in your component.
// We don't actually use this in the example code,
// but include the ref for illustration.
const ferrostarMapRef: Ref<FerrostarMap> = ref(null)

// Create a search box,
// which we will configure to show when navigation is not active.
const searchBox = new MapLibreSearchControl({
  onResultSelected: async (feature) => {
    const ferrostar = ferrostarMapRef.value;
    const coordinates = feature.geometry.coordinates;
    const waypoints = [{ coordinate: { lat: coordinates[1], lng: coordinates[0] }, kind: "Break" }];

    const location = await ferrostar.locationProvider.getCurrentLocation(30_000);
    // Use the acquired user location to request the route
    const routes = await ferrostar.getRoutes(location, waypoints);
    const route = routes[0];

    // Start the navigation
    ferrostar.startNavigation(route, {
      stepAdvance: {
        RelativeLineStringDistance: {
          minimumHorizontalAccuracy: 25,
          automaticAdvanceDistance: 10,
        },
      },
      routeDeviationTracking: {
        StaticThreshold: {
          minimumHorizontalAccuracy: 25,
          maxAcceptableDeviation: 10.0,
        },
      },
      snappedLocationCourseFiltering: "Raw",
    });
  },
});

const locationProvider = new BrowserLocationProvider();

onMounted(() => {
  // More complex properties should be set here.
  // Web components fundamentally use string-based attributes,
  // so while some of these *can* work as v-bind props,
  // it's best to do it like this.
  ferrostarMapRef.value.locationProvider = locationProvider;
  ferrostarMapRef.value.configureMap = (map) => map.addControl(searchBox, 'top-left');
  ferrostarMapRef.value.onNavigationStart = (map) => map.removeControl(searchBox);
  ferrostarMapRef.value.onNavigationStop = (map) => map.addControl(searchBox, 'top-left');
});
</script>

<template>
  <ferrostar-map
      ref="ferrostarMapRef"
      valhallaEndpointUrl="https://api.stadiamaps.com/route/v1"
      styleUrl="https://tiles.stadiamaps.com/styles/outdoors.json"
      profile="bicycle"
      :center="{lng: 126.8, lat: 37.6}"
      :zoom=6
      :pitch="45"
      :customStyles="searchBoxStyle"
  ></ferrostar-map>
</template>

<style scoped>
ferrostar-map {
  display: block;
  width: 100%;
  height: 100%;
}
</style>
