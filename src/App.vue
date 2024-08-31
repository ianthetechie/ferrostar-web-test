<script setup>
import {FerrostarMap} from "@stadiamaps/ferrostar-webcomponents";
import {onMounted, ref} from "vue";
import {GeolocateControl} from "maplibre-gl";

const map = ref(null)

onMounted(() => {
  setTimeout(() => {
    map.value.map.addControl(new GeolocateControl({
      positionOptions: {
        enableHighAccuracy: true
      },
      trackUserLocation: true
    }))
  }, 100)
})
</script>

<template>
  <ferrostar-map
      ref="map"
      valhallaEndpointUrl="https://api.stadiamaps.com/route/v1"
      styleUrl="https://tiles.stadiamaps.com/styles/outdoors.json"
      profile="bicycle"
      :center="{lng: 126.8, lat: 37.6}"
      :zoom=11
      :pitch="45"
      :useIntegratedSearchBox=true
  ></ferrostar-map>
</template>

<style scoped>
ferrostar-map {
  display: block;
  width: 100%;
  height: 100%;
}
</style>
