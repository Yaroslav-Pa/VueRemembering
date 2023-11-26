<script>
import { ref } from 'vue';
import 'leaflet/dist/leaflet.css';
import L from 'leaflet';

export default {
  setup() {
    return {
      map: null,
      filterType: null,
      layersGroup: new L.LayerGroup(),
      data: [],
      smellsTypes: [],
    };
  },
  async mounted() {
    this.createMap();
    await this.fetchData();
    this.placeMarkersByData();
    this.smellTypeGetter();
    this.$forceUpdate(); // there are some problem with updating smellsTypes[] and if I dont do forceUpdate it will not update a select data
  },
  methods: {
    createMap() {
      this.map = L.map('map').setView([47.834264774079, 35.14758198304963], 11);
      L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
        maxZoom: 19,
        attribution:
          '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>',
      }).addTo(this.map);
    },
    async fetchData() {
      try {
        const response = await fetch('https://zpspace.com.ua/api/airdata', {
          method: 'GET',
        });
        this.data = await response.json();
      } catch (err) {
        console.error(err);
      }
    },
    placeMarkersByData(filteredData = this.data) {
      this.map.removeLayer(this.layersGroup);
      this.layersGroup = new L.LayerGroup();
      filteredData.forEach((element) => {
        L.marker([element.latitude, element.longitude]).addTo(this.layersGroup);
      });
      this.map.addLayer(this.layersGroup);
    },

    placeFilteredMarkers(filterType = null) {
      if (this.smellsTypes.includes(filterType) || filterType === 'none') {
        const filteredData = this.dataFilter(filterType);
        this.placeMarkersByData(filteredData);
      } else if (filterType === null) {
        this.placeMarkersByData();
      }
    },

    dataFilter(smellName) {
      let filteredData = null;
      if (smellName !== 'none') {
        filteredData = this.data.filter(
          (elem) => elem.kind_of_smell === smellName
        );
      } else {
        filteredData = this.data.filter(
          (elem) => !this.smellsTypes.includes(elem.kind_of_smell)
        );
      }

      return filteredData;
    },
    smellTypeGetter() {
      this.data.forEach((elem) => {
        if (
          elem.kind_of_smell &&
          typeof elem.kind_of_smell === 'string' &&
          !this.smellsTypes.includes(elem.kind_of_smell)
        ) {
          this.smellsTypes.push(elem.kind_of_smell);
        }
      });
    },
  },
};
</script>

<template>
  <section class="flex flex-col gap-5">
    <div
      id="map"
      class="h-[600px] w-[300px] md:h-[400px] md:w-[500px] lg:h-[500px] lg:w-[800px]"
    ></div>
    <h3>Фільтр смороду:</h3>
    <select
      v-if="smellsTypes.length > 0"
      id="filter"
      class="p-3 border-2 border-black"
      v-model="filterType"
      @change="placeFilteredMarkers(filterType)"
    >
      <option :value="null">Без фільтру</option>
      <option value="none">Не визначено</option>
      <option v-for="smell in this.smellsTypes" :value="`${smell}`">{{ smell }}</option>
    </select>
    <div v-else>
      Зачекайте, дані ще завантажуються...
    </div>
  </section>
</template>

<style>
</style>
