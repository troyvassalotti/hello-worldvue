<script>
/*
TODO
- Better handling of multiple files, i.e. removing old layers or mappings when a new file is added.
 */
import "leaflet/dist/leaflet.css"
import L from "leaflet"
import "leaflet-kmz/dist/leaflet-kmz"
import "leaflet-gpx/gpx"
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
    // Create the initial Leaflet map
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
      // Check to see if there are any Object URLs to revoke
      if (this.currentObjectURLs.length > 0) {
        for (let obj = 0; obj < this.currentObjectURLs.length; obj++) {
          URL.revokeObjectURL(this.currentObjectURLs[obj])
        }
        this.currentObjectURLs = [] // reset the objectURLs array since we are only handling one file at the moment
      }

      // Set some variables for use later
      const fileList = el.files
      const firstFile = fileList[0]
      const objectURL = URL.createObjectURL(firstFile)

      this.currentObjectURLs.push(objectURL)

      if (firstFile.type === "application/gpx+xml") {
        // Parse GPX files
        let gpx = objectURL
        new L.GPX(gpx, {async: true}).on('loaded', e => {
          this.mapDiv.fitBounds(e.target.getBounds())
        }).addTo(this.mapDiv)
      } else {
        // Parse KML and KMZ files
        let kmz = L.kmzLayer().addTo(this.mapDiv)
        let control = L.control.layers(null, null, {collapsed: false}).addTo(this.mapDiv)
        kmz.on('load', e => {
          control.addOverlay(e.layer, e.name)
        })
        kmz.load(objectURL)
      }
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
      <label for="upload">Upload Your Map <input id="upload" type="file" accept=".kmz, .kml, .gpx" @change="parseFile($event.target)"></label>
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
