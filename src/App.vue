<script setup lang="ts">
import {
  BrowserLocationProvider,
  FerrostarMap
} from "@stadiamaps/ferrostar-webcomponents";
import {nextTick, onMounted, Ref, ref} from "vue";
import {MapLibreSearchControl} from "@stadiamaps/maplibre-search-box";
import {GeolocateControl, Map} from "maplibre-gl";
import searchBoxStyle from "@stadiamaps/maplibre-search-box/dist/style.css?inline";

// If you need very fine-grained control over the component,
// you can use the Vue `ref` to keep a reference and react to state changes
// elsewhere in your component.
// We don't actually use this in the example code,
// but include the ref for illustration.
const ferrostarMapRef: Ref<FerrostarMap> = ref(null)

// Similarly, you can get fine-grained control over the MapLibre map
// by creating it yourself and providing a container
const mapContainer = ref(null)

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

// In the case that you create + pass in your own MapLibre Map,
// it's helpful to have the style as global state since it's required at init
// on both the map and the web component (the web component can observe changes)
const styleUrl = "https://tiles.stadiamaps.com/styles/outdoors.json";
// const styleUrl = "https://demotiles.maplibre.org/style.json";

onMounted(() => {
  // More complex properties should be set here.
  // Web components fundamentally use string-based attributes,
  // so while some of these *can* work as v-bind props,
  // it's best to do it like this.
  ferrostarMapRef.value.locationProvider = locationProvider;
  // ferrostarMapRef.value.configureMap = (map) => map.addControl(searchBox, 'top-left');
  ferrostarMapRef.value.onNavigationStart = (map) => map.removeControl(searchBox);
  ferrostarMapRef.value.onNavigationStop = (map) => map.addControl(searchBox, 'top-left');

  // This appears to be necessary for the web component to init styles?
  nextTick(() => {
    const map =
        new Map({
          container: mapContainer.value,
          style: styleUrl,
          center: [0, 0],
          zoom: 4,
          bearing: 0,
          pitch: 45,
          attributionControl: { compact: true },
        });

    map.on("load", (e) => {
      console.log("map loaded", map);
      map.addControl(searchBox, 'top-left');
      map.addControl(new GeolocateControl({
        positionOptions: {
          enableHighAccuracy: true,
        },
        trackUserLocation: true,
      }))
    });

    ferrostarMapRef.value.map = map;
  });
});
</script>

<template>
  <ferrostar-map
      ref="ferrostarMapRef"
      valhallaEndpointUrl="https://api.stadiamaps.com/route/v1"
      :styleUrl="styleUrl"
      profile="bicycle"
      :center="{lng: 126.8, lat: 37.6}"
      :zoom=6
      :pitch="45"
      :customStyles="searchBoxStyle"
  >
    <div ref="mapContainer"></div>
  </ferrostar-map>
</template>

<style scoped>
@import 'maplibre-gl/dist/maplibre-gl.css';
@import "@stadiamaps/maplibre-search-box/dist/style.css";

ferrostar-map {
  display: block;
  width: 100%;
  height: 100%;
}
</style>
