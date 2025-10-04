<template>
  <div
    ref="mapElement"
    :style="{
      width: `${width}px`,
      height: `${height}px`,
      maxWidth: '100%',
    }"
  />
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { setOptions, importLibrary } from '@googlemaps/js-api-loader'

const GOOGLE_MAP_API_KEY = 'AIzaSyC72ke4z-JR-oJFKuGcb-cUYw3Yhp5_Vv0'
const mapElement = ref(null)
let map = null

defineOptions({
  name: 'GoogleMapComponent',
})

const props = defineProps({
  markers: {
    type: Array,
    default: () => [],
  },
  zoom: {
    type: Number,
    default: 12,
  },
  center: {
    type: Object,
    default: () => ({
      lat: 36.2972,
      lng: 59.6067,
    }),
  },
  width: {
    type: Number,
    default: 375,
  },
  height: {
    type: Number,
    default: 400,
  },
})

onMounted(async () => {
  setOptions({
    key: GOOGLE_MAP_API_KEY,
  })

  const { Map } = await importLibrary('maps')
  map = new Map(mapElement.value, {
    center: props.center,
    zoom: props.zoom,
  })

  if (props.markers.length > 0)
    props.markers.forEach(marker => {
      new google.maps.Marker({
        map,
        position: { lat: marker.lat, lng: marker.lng },
        icon: {
          path: google.maps.SymbolPath.FORWARD_CLOSED_ARROW,
          fillColor: 'blue',
          fillOpacity: 0.6,
          strokeWeight: 0,
          rotation: marker.heading,
          scale: 10,
        },
      })
    })
})
</script>
