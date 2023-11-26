<script >
import { ref } from 'vue'
import "leaflet/dist/leaflet.css";
import L from "leaflet";

export default {
  props: ['msg'],
  setup(props) {
    return {
      count: ref(0),
      map: null,
      data: [],
    }
  },
  mounted() {
    this.map = L.map('map').setView([47.834264774079, 35.14758198304963], 11);
    L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
        maxZoom: 19,
        attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
    }).addTo(this.map);
  }
}
</script>

<template>
  <section class="flex flex-col gap-5">
    <div id="map" style="height: 500px; width: 500px;">
  </div>
  <input type='' class="px-5 py-3 bg-white"/>
  </section>

</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
