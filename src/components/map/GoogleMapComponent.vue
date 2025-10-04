<template>
  <div ref="mapElement" style="width: 375px; max-width: 100%; height: 400px" />
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { setOptions, importLibrary } from '@googlemaps/js-api-loader'

const GOOGLE_MAP_API_KEY = 'AIzaSyC72ke4z-JR-oJFKuGcb-cUYw3Yhp5_Vv0'
const mapElement = ref(null)
let map = null

onMounted(async () => {
  setOptions({
    key: GOOGLE_MAP_API_KEY,
  })

  const { Map } = await importLibrary('maps')
  map = new Map(mapElement.value, {
    center: { lat: 36.2972, lng: 59.6067 },
    zoom: 12,
  })
})
</script>
