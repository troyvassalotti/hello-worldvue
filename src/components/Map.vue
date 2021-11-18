<script>
import "leaflet/dist/leaflet.css"
import L from "leaflet"
import "leaflet-kmz/dist/leaflet-kmz"
import {mapboxAccessKey} from "../mapbox";

export default {
  name: "Map",
  data() {
    return {
      mapDiv: '',
      center: [40, 0],
      currentObjectURLs: []
    }
  },
  methods: {
    setupLeafletMap: function () {
      this.mapDiv = L.map('map', {preferCanvas: true})
          .setView(this.center, 2)
          // .locate({setView: true})
      L.tileLayer(`https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={accessToken}`, {
        attribution: 'Map data (c) <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery (c) <a href="https://www.mapbox.com/">Mapbox</a>',
        maxZoom: 18,
        id: "mapbox/streets-v11",
        accessToken: mapboxAccessKey,
      }).addTo(this.mapDiv)
    },
    parseFile: function (el) {
      if (this.currentObjectURLs.length > 0) {
        for (let obj = 0; obj < this.currentObjectURLs.length; obj++) {
          URL.revokeObjectURL(this.currentObjectURLs[obj])
        }
      }
      const fileList = el.files
      const objectURL = URL.createObjectURL(fileList[0])
      this.currentObjectURLs.push(objectURL)
      let kmz = L.kmzLayer().addTo(this.mapDiv)
      let control = L.control.layers(null, null, {collapsed: false}).addTo(this.mapDiv)
      kmz.on('load', e => {
        control.addOverlay(e.layer, e.name)
      })
      kmz.load(objectURL)
    }
  },
  mounted() {
    this.setupLeafletMap()
  },
}
</script>

<template>
  <main>
    <form enctype="multipart/form-data" novalidate>
      <label for="upload">Upload Your Map <input id="upload" type="file" accept=".kmz, .kml" @change="parseFile($event.target)"></label>
    </form>
    <div id="map"></div>
  </main>
</template>

<style scoped>
#map {
  block-size: clamp(10rem, 50vh, 60rem);
}

label {
  display: inline-flex;
  flex-direction: column;
  gap: 1rem;
}
</style>
