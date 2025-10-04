<template>
  <q-page class="flex flex-center">
    <div class="column items-center q-gutter-md">
      <h4>Google Map</h4>
      <google-map-component :markers="markers" />
    </div>
  </q-page>
</template>

<script setup>
import GoogleMapComponent from 'src/components/map/GoogleMapComponent.vue'

defineOptions({
  name: 'IndexPage',
})

const markers = [
  { lat: 36.2972, lng: 59.6067, heading: 0 },
  { lat: 36.3035, lng: 59.582, heading: 90 },
  { lat: 36.286, lng: 59.6157, heading: 180 },
  { lat: 36.31, lng: 59.63, heading: 270 },
]
</script>
