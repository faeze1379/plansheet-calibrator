<template>
  <q-page class="flex flex-center">
    <div class="column items-center q-gutter-md">
      <h4>Google Map</h4>
      <google-map-component />
    </div>
  </q-page>
</template>

<script setup>
import GoogleMapComponent from 'src/components/map/GoogleMapComponent.vue'
defineOptions({
  name: 'IndexPage',
})
</script>
