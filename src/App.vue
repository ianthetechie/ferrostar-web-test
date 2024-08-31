<script setup>
import {FerrostarMap} from "@stadiamaps/ferrostar-webcomponents";
</script>

<template>
  <ferrostar-map
      id="core"
      valhallaEndpointUrl="https://api.stadiamaps.com/route/v1"
      styleUrl="https://tiles.stadiamaps.com/styles/outdoors.json"
      profile="bicycle"
      :center="{lng: -122.42, lat: 37.81}"
      :zoom=6
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
