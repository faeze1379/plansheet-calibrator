<template>
  <div>
    <div class="flex items-center justify-between q-pa-md">
      <b class="text-h6" v-if="planSheet">Plan Sheet: {{ planSheet.name }}</b>
      <q-btn
        class="q-mb-md"
        label="Calibrate"
        color="orange"
        no-caps
        @click="calibrate"
      />
    </div>
    <div
      ref="mapElement"
      :style="{
        width: `100vw`,
        height: `90vh`,
      }"
    />
  </div>
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
  planSheets: {
    type: Array,
    default: () => [],
  },
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
})

const planSheet = ref(null)
const markersShown = ref([])

async function getOverlayBounds(planSheetData) {
  const { topLeft, topRight, bottomRight, bottomLeft } =
    planSheetData.mapOverLay.planSheetBound

  const lats = [
    topLeft.latitude,
    topRight.latitude,
    bottomRight.latitude,
    bottomLeft.latitude,
  ]
  const lngs = [
    topLeft.longitude,
    topRight.longitude,
    bottomRight.longitude,
    bottomLeft.longitude,
  ]
  const sw = { lat: Math.min(...lats), lng: Math.min(...lngs) }
  const ne = { lat: Math.max(...lats), lng: Math.max(...lngs) }

  return { sw, ne }
}

function calibrate() {
  markersShown.value = props.markers.filter(
    marker => marker.currentLayout == planSheet.value.uniqueId
  )

  if (markersShown.value.length > 0)
    markersShown.value.forEach(marker => {
      new google.maps.Marker({
        map,
        position: { lat: marker.latitude, lng: marker.longitude },
        icon: {
          path: google.maps.SymbolPath.FORWARD_CLOSED_ARROW,
          fillColor: '#0078ff',
          fillOpacity: 1,
          strokeWeight: 1,
          rotation: marker.angle,
          scale: 8,
        },
      })
    })

  console.log('markersShown', markersShown.value)
}

async function loadMap() {
  setOptions({
    key: GOOGLE_MAP_API_KEY,
  })

  if (props.planSheets.length > 0) {
    planSheet.value = props.planSheets.find(
      plan => plan.mapOverLay != null && plan.url
    )
    const { sw, ne } = await getOverlayBounds(planSheet.value)
    const { Map } = await importLibrary('maps')

    map = new Map(mapElement.value, {
      center: sw,
      zoom: props.zoom,
    })

    const bounds = new google.maps.LatLngBounds(sw, ne)
    const overlay = new google.maps.GroundOverlay(planSheet.value.url, bounds)

    overlay.setMap(map)

    map.fitBounds(bounds, { padding: 0 })
  }
}

onMounted(async () => await loadMap())
</script>
