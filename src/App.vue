<template>
  <main>
    <div id="form-main">
      <h1>🖼 Maplibre screenshotter</h1>
      <label for="styles">Styles:</label>
      <div class="form-styles-line" v-for="(style, index) in styles" :key="index">
        <input type="text" :id="'style-' + index" v-model="styles[index]" />
        <button type="button" @click="styles.splice(index, 1)">❌</button>
      </div>
      <button type="button" @click="styles.push('')">Add Style ＋</button>
      <div class="form-field">
        <label for="width">Width:</label>
        <input type="number" id="width" v-model="width" />
      </div>

      <div class="form-field">
        <label for="height">Height:</label>
        <input type="number" id="height" v-model="height" />
      </div>

      <div class="form-field">
        <label for="lng">Longitude:</label>
        <input type="number" id="lng" v-model="lng" step="0.01" />
      </div>

      <div class="form-field">
        <label for="lat">Latitude:</label>
        <input type="number" id="lat" v-model="lat" step="0.01" />
      </div>

      <div class="form-field">
        <label for="zoom">Zoom:</label>
        <input type="number" id="zoom" v-model="zoom" />
      </div>
      <button type="button" @click="downloadMap">Generate screenshots</button>
    </div>
    <MapComponent
      v-if="styles.length > 0 && styles[0].length > 0"
      ref="mapComponent"
      :styles="styles"
      :width="width"
      :height="height"
      :coordinates="[lng, lat]"
      :zoom="zoom"
      @on-move="
        (coord) => {
          lng = coord.lng
          lat = coord.lat
        }
      "
      @on-zoom="(z) => (zoom = z)"
      @on-resize="
        (size) => {
          width = size.width
          height = size.height
        }
      "
      @on-screenshot="(screenshot) => screenshots.push(screenshot)"
    />
  </main>
  <div id="screenshots">
    <img v-for="screenshot in screenshots" :key="screenshot" :src="screenshot" />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import MapComponent from './components/MapComponent.vue'

const mapComponent = ref(null)

const styles = ref([])
const width = ref(400)
const height = ref(400)
const lng = ref(1.45)
const lat = ref(43.6)
const zoom = ref(10)
const screenshots = ref([])

const downloadMap = () => {
  mapComponent.value.downloadMap()
}
</script>

<style scoped>
#form-main {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: stretch;
  gap: 1em;
  width: 270px;
  margin-bottom: 2em;
  height: 100%;
  text-align: left;
}
.form-field {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  width: 100%;
}
#form-styles {
  display: flex;
  flex-direction: column;
  gap: 0.5em;
  width: 270px;
  margin-bottom: 2em;
}
.form-styles-line {
  width: 100%;
  display: flex;
  gap: 0.5em;
}
.form-styles-line input {
  flex: 1;
}
main {
  display: flex;
  flex-direction: row;
  gap: 2em;
}
#screenshots {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  gap: 10px;
  margin-top: 30px;
  border-top: 1px solid var(--color-border);
  padding-top: 2em;
  align-items: flex-start;
}
main input {
  border-radius: 6px;
  border: 1px solid var(--color-border);
}
h1 {
  font-size: 2em;
  text-align: left;
}
</style>
