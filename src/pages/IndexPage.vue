<template>
  <q-page class="flex flex-center">
    <div class="column items-center q-gutter-md">
      <google-map-component :markers="markers" :planSheets="planSheets" />
    </div>
  </q-page>
</template>

<script setup>
import { ref } from 'vue'
import GoogleMapComponent from 'src/components/map/GoogleMapComponent.vue'
import markersData from 'src/fake-data/markers.json'
import planSheetsData from 'src/fake-data/plan-sheets.json'

defineOptions({
  name: 'IndexPage',
})

const planSheets = ref(planSheetsData)
const markers = ref(markersData)
</script>
