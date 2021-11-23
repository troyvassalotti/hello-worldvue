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
      currentObjectURLs: [],
      showGPXStats: false,
      gpxStats: []
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
        for (const url of this.currentObjectURLs) {
          URL.revokeObjectURL(url)
        }
        this.currentObjectURLs = [] // reset the objectURLs array since we are only handling one file at the moment
        this.gpxStats = [] // reset the GPX stats array since we are only accounting for one track right now
      }

      // Set some variables for use later
      const fileList = el.files

      for (const file of fileList) {
        let objectURL = URL.createObjectURL(file)
        this.currentObjectURLs.push(objectURL)

        if (file.type === "application/gpx+xml") {
          // Show GPX Stats
          this.showGPXStats = true
          // Parse GPX files
          new L.GPX(objectURL, {async: true}).on('loaded', e => {
            let el = e.target
            let stats = this.gpxStats
            this.mapDiv.fitBounds(el.getBounds())

            // Grab the total distance
            let distance = el.get_distance_imp()
            stats.push({
              type: "Total Distance",
              value: distance,
              unit: "miles"
            })

            // Grab the avg speed
            let movingSpeed = el.get_moving_speed_imp()
            stats.push({
              type: "Average Moving Speed",
              value: movingSpeed,
              unit: "mph"
            })

            // Grab the max speed
            let maxSpeed = el.get_speed_max_imp()
            stats.push({
              type: "Max Speed",
              value: maxSpeed,
              unit: "mph"
            })

            // Grab the elevation gain
            let elevGain = el.get_elevation_gain_imp()
            stats.push({
              type: "Elevation Gain",
              value: elevGain,
              unit: "ft"
            })
          }).addTo(this.mapDiv)
        } else {
          // Don't show GPX stats
          this.showGPXStats = false
          // Parse KML and KMZ files
          let kmz = L.kmzLayer().addTo(this.mapDiv)
          let control = L.control.layers(null, null, {collapsed: false}).addTo(this.mapDiv)
          kmz.on('load', e => {
            control.addOverlay(e.layer, e.name)
          })
          kmz.load(objectURL)
        }
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
      <label for="upload">Upload Your Map <input id="upload" type="file" multiple accept=".kmz, .kml, .gpx" @change="parseFile($event.target)"></label>
    </form>
    <div id="map"></div>
    <section class="gpxStats" v-show="showGPXStats">
      <header>
        <h2>Track Stats</h2>
      </header>
      <div class="content">
        <ul>
          <li v-for="stat in gpxStats">
            <span class="stat-type">{{ stat.type }}</span>: {{ stat.value }} {{ stat.unit }}
          </li>
        </ul>
      </div>
    </section>
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

.gpxStats {
  margin-block-start: 5rem;
}

.stat-type {
  font-weight: bold;
}
</style>
