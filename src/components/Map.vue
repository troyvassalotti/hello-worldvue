<script>
import "leaflet/dist/leaflet.css";
import L from "leaflet";
import "leaflet-kmz/dist/leaflet-kmz";
import "leaflet-gpx/gpx";
import mapboxAccessKey from "../mapbox";

export default {
  name: "Map",
  data() {
    return {
      map: "",
      currentObjectURLs: [],
      showGPXStats: false,
      gpxStats: new Map(),
      trackLayerGroup: L.layerGroup(),
    };
  },
  methods: {
    // Create the initial Leaflet map
    setupLeafletMap: function () {
      this.map = L.map("map", { preferCanvas: true }).setView([40, 0], 2);

      L.tileLayer(
        `https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={accessToken}`,
        {
          accessToken: mapboxAccessKey,
          attribution:
            'Map data (c) <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery (c) <a href="https://www.mapbox.com/">Mapbox</a>',
          id: "mapbox/streets-v11",
          maxZoom: 18,
        },
      ).addTo(this.map);
    },
    reset: function () {
      if (this.currentObjectURLs.length > 0) {
        for (const url of this.currentObjectURLs) {
          URL.revokeObjectURL(url);
        }

        this.currentObjectURLs = []; // reset the objectURLs array since we are only handling one upload at the moment
        this.gpxStats = new Map(); // reset the GPX stats array since we are only accounting for one upload right now
        this.trackLayerGroup.clearLayers();
      }
    },
    createGPXStat: function (type, value, unit) {
      return {
        type,
        value: typeof value === "number" ? value.toFixed(2) : value,
        unit,
      };
    },
    parseFile: function (el) {
      this.reset();

      const { files } = el;
      let map = this.map;

      for (const file of files) {
        let objectURL = URL.createObjectURL(file);
        this.currentObjectURLs.push(objectURL);

        if (file.type === "application/gpx+xml") {
          this.showGPXStats = true;

          new L.GPX(objectURL, { async: true })
            .on("loaded", (event) => {
              let { target } = event;

              map.fitBounds(target.getBounds());

              let name = target.get_name();

              let totalTime = this.createGPXStat(
                "Total Time",
                target.get_duration_string_iso(target.get_total_time(), true),
                "",
              );

              let distance = this.createGPXStat(
                "Total Distance",
                target.get_distance_imp(),
                "miles",
              );

              let avgSpeed = this.createGPXStat(
                "Average Moving Speed",
                target.get_moving_speed_imp(),
                "mph",
              );

              let avgPace = this.createGPXStat(
                "Average Moving Pace",
                target.get_duration_string_iso(
                  target.get_moving_pace_imp(),
                  true,
                ),
                "",
              );

              let elevationLoss = this.createGPXStat(
                "Elevation Loss",
                target.get_elevation_loss_imp(),
                "ft",
              );

              let elevationGain = this.createGPXStat(
                "Elevation Gain",
                target.get_elevation_gain_imp(),
                "ft",
              );

              let elevationMax = this.createGPXStat(
                "Elevation Max",
                target.get_elevation_max_imp(),
                "ft",
              );

              let elevationMin = this.createGPXStat(
                "Elevation Min",
                target.get_elevation_min_imp(),
                "ft",
              );

              let stats = [
                totalTime,
                distance,
                avgSpeed,
                avgPace,
                elevationGain,
                elevationLoss,
                elevationMax,
                elevationMin,
              ];

              this.gpxStats.set(name, stats);
            })
            .addTo(this.trackLayerGroup);
        } else {
          this.showGPXStats = false;
          let kmz = L.kmzLayer().addTo(map);
          let control = L.control
            .layers(null, null, { collapsed: false })
            .addTo(map);

          kmz.on("load", (e) => {
            control.addOverlay(e.layer, e.name);
          });

          kmz.load(objectURL);
        }

        this.trackLayerGroup.addTo(map);
      }
    },
  },
  mounted() {
    this.setupLeafletMap();
  },
};
</script>

<template>
  <section id="viewer">
    <form enctype="multipart/form-data" novalidate>
      <label for="upload"
        >Upload Your Map
        <input
          id="upload"
          type="file"
          multiple
          accept=".kmz, .kml, .gpx"
          @change="parseFile($event.target)"
      /></label>
    </form>
    <div id="map"></div>
  </section>
  <section id="gpxStats" v-show="showGPXStats">
    <header>
      <h2>Track Stats</h2>
    </header>
    <ul>
      <li v-for="[name, stats] in gpxStats">
        <strong>{{ name }}</strong>
        <ul>
          <li v-for="stat in stats">
            <strong>{{ stat.type }}</strong
            >: {{ stat.value }} {{ stat.unit }}
          </li>
        </ul>
      </li>
    </ul>
  </section>
</template>

<style scoped>
#map {
  block-size: clamp(10rem, 50vh, 60rem);
}
</style>
