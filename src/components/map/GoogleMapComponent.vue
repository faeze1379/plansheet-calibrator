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
    default: () => ({}),
  },
})

const planSheet = ref(null)
const overlayBounds = ref(null)
const overlayImageSize = ref({ width: 0, height: 0 })
const drawnMarkers = []

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

function clearMarkers() {
  while (drawnMarkers.length) {
    const m = drawnMarkers.pop()
    m.setMap(null)
  }
}

function loadMarkersGeo(markers) {
  clearMarkers()
  markers.forEach(marker => {
    const m = new google.maps.Marker({
      map,
      position: { lat: marker.latitude, lng: marker.longitude },
      icon: {
        path: google.maps.SymbolPath.FORWARD_CLOSED_ARROW,
        fillColor: '#0078ff',
        fillOpacity: 1,
        strokeWeight: 1,
        rotation: marker.angle || 0,
        scale: 6,
      },
    })
    drawnMarkers.push(m)
  })
}

function bilinearLatLng(u, v, corners) {
  const { topLeft: TL, topRight: TR, bottomRight: BR, bottomLeft: BL } = corners
  const lat =
    (1 - u) * (1 - v) * TL.latitude +
    u * (1 - v) * TR.latitude +
    u * v * BR.latitude +
    (1 - u) * v * BL.latitude

  const lng =
    (1 - u) * (1 - v) * TL.longitude +
    u * (1 - v) * TR.longitude +
    u * v * BR.longitude +
    (1 - u) * v * BL.longitude

  return { lat, lng }
}

function loadMarkersByPixel(markers) {
  if (
    !planSheet.value ||
    !overlayImageSize.value.width ||
    !overlayImageSize.value.height
  )
    return

  const corners = planSheet.value.mapOverLay.planSheetBound
  clearMarkers()

  markers.forEach(marker => {
    const u = marker.px / overlayImageSize.value.width
    const v = marker.py / overlayImageSize.value.height

    const { lat, lng } = bilinearLatLng(u, v, corners)

    const m = new google.maps.Marker({
      map,
      position: { lat, lng },
      icon: {
        path: google.maps.SymbolPath.FORWARD_CLOSED_ARROW,
        fillColor: '#0078ff',
        fillOpacity: 1,
        strokeWeight: 1,
        rotation: marker.angle || 0,
        scale: 6,
      },
    })
    drawnMarkers.push(m)
  })
}

function calibrate() {
  const selected = props.markers.filter(
    marker => marker.currentLayout == planSheet.value.uniqueId
  )

  const hasPixelCoords =
    selected.length > 0 &&
    typeof selected[0]?.px === 'number' &&
    typeof selected[0]?.py === 'number'

  if (hasPixelCoords) loadMarkersByPixel(selected)
  else loadMarkersGeo(selected)
}

async function preloadOverlayImage(url) {
  await new Promise((resolve, reject) => {
    const img = new Image()
    img.onload = () => {
      overlayImageSize.value = {
        width: img.naturalWidth,
        height: img.naturalHeight,
      }
      resolve()
    }
    img.onerror = reject
    img.src = url
  })
}

async function loadMap() {
  setOptions({ key: GOOGLE_MAP_API_KEY })

  if (props.planSheets.length > 0) {
    planSheet.value = props.planSheets.find(
      plan => plan.mapOverLay != null && plan.url
    )
    if (!planSheet.value) return

    await preloadOverlayImage(planSheet.value.url)

    const { sw, ne } = await getOverlayBounds(planSheet.value)
    overlayBounds.value = { sw, ne }

    const { Map } = await importLibrary('maps')

    map = new Map(mapElement.value, {
      center: sw,
      zoom: props.zoom,
      gestureHandling: 'greedy',
      mapTypeControl: false,
      fullscreenControl: false,
      streetViewControl: false,
    })

    const bounds = new google.maps.LatLngBounds(sw, ne)
    const overlay = new google.maps.GroundOverlay(planSheet.value.url, bounds)
    overlay.setOpacity(1)
    overlay.setMap(map)
    map.fitBounds(bounds, { padding: 0 })
  }
}

onMounted(async () => {
  await loadMap()
  await loadMarkersGeo(props.markers)
})
</script>
