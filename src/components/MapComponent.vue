<template>
  <div id="map-wrapper">
    <div
      id="map"
      ref="mapContainer"
      :style="{ height: `${props.height}px`, width: `${props.width}px` }"
    ></div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import maplibregl from 'maplibre-gl'
import 'maplibre-gl/dist/maplibre-gl.css'

const props = defineProps({
  styles: {
    type: Array,
    required: true
  },
  width: {
    type: Number,
    required: true
  },
  height: {
    type: Number,
    required: true
  },
  coordinates: {
    type: String,
    required: true
  },
  zoom: {
    type: Number,
    required: true
  }
})

const emit = defineEmits(['onMove', 'onZoom', 'onResize', 'onScreenshot'])

const mapContainer = ref(null)
const map = ref(null)

onMounted(() => {
  map.value = new maplibregl.Map({
    container: mapContainer.value,
    preserveDrawingBuffer: true,
    style: props.styles[0],
    center: props.coordinates,
    zoom: props.zoom
  })
  map.value.on('moveend', () => {
    const [lng, lat] = map.value.getCenter().toArray()
    emit('onMove', { lng, lat })
  })
  map.value.on('zoomend', () => {
    const zoom = map.value.getZoom()
    emit('onZoom', zoom)
  })
  map.value.on('resize', () => {
    emit('onResize', { width: map.value.transform.width, height: map.value.transform.height })
  })
})

watch(props, (newProps) => {
  map.value.setCenter(newProps.coordinates)
  map.value.setZoom(newProps.zoom)
})

const downloadMap = () => {
  const screenshotStyle = (styleIndex) => {
    const style = props.styles[styleIndex];
    map.value.setStyle(style, { diff: false })
    map.value.once('idle', () => {
      map.value.getCanvas().toBlob((blob) => {
        const url = URL.createObjectURL(blob)
        emit('onScreenshot', url)
        if (props.styles[styleIndex + 1]) screenshotStyle(styleIndex + 1)
        else map.value.setStyle(props.styles[0])
      })
    })
  }

  screenshotStyle(0)
}
defineExpose({
  downloadMap
})
</script>

<style scoped>
#map {
  resize: both;
  overflow: hidden;
}
#map-wrapper {
  border-left: 1px solid var(--color-border);
  padding-left: 2em;
}
</style>
