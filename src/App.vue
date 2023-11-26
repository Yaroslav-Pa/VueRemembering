<script>
import { ref } from 'vue';
import 'leaflet/dist/leaflet.css';
import L from 'leaflet';

export default {
  props: ['msg'],
  setup(props) {
    return {
      count: ref(0),
      map: null,
      filterType: null,
      data: [],
      layersGroup: new L.LayerGroup(),
    };
  },
  async mounted() {
    this.createMap();
    await this.fetchData();
    this.placeMarkersByData();
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
    placeFilteredMarkers(filterType = null) {
      console.log('changedFilter');
      switch (filterType) {
        case 'null': {
          this.placeMarkersByData();
          break;
        }
        case 'iodine': {
          const filteredData = this.dataFilter('Йод');
          this.placeMarkersByData(filteredData);
          break;
        }
        case 'rot': {
          const filteredData = this.dataFilter('Гниль');
          this.placeMarkersByData(filteredData);
          break;
        }
        case 'plastic': {
          const filteredData = this.dataFilter('Горілий пластик');
          this.placeMarkersByData(filteredData);
          break;
        }
        case 'metal': {
          const filteredData = this.dataFilter('Металургійний гар');
          this.placeMarkersByData(filteredData);
          break;
        }
        case 'chemistry': {
          const filteredData = this.dataFilter('Хімія');
          this.placeMarkersByData(filteredData);
          break;
        }
        case 'sulfide': {
          const filteredData = this.dataFilter('Сірководень');
          this.placeMarkersByData(filteredData);
          break;
        }
      }
    },
    placeMarkersByData(filteredData = this.data) {
      this.map.removeLayer(this.layersGroup);
      this.layersGroup = new L.LayerGroup();
      filteredData.forEach((element) => {
          L.marker([element.latitude, element.longitude]).addTo(this.layersGroup);
      });
      this.map.addLayer(this.layersGroup)
    },
    dataFilter(smellName){
      const filteredData = this.data.filter(
            (elem) => elem.kind_of_smell === smellName
      );
      return filteredData;
    }
  },
};
</script>

<template>
  <section class="flex flex-col gap-5">
    <div id="map" class="h-[600px] w-[300px] md:h-[400px] md:w-[500px] lg:h-[500px] lg:w-[800px]"></div>
    <h3>Фільтр смороду:</h3>
    <select
      id="filter"
      class="p-3 border-2 border-black"
      v-model="this.filterType"
      @change="placeFilteredMarkers(this.filterType)"
    >
      <option value="null">Без фільтру</option>
      <option value="iodine">Йод</option>
      <option value="rot">Гниль</option>
      <option value="plastic">Горілий пластик</option>
      <option value="metal">Металургійний гар</option>
      <option value="chemistry">Хімія</option>
      <option value="sulfide">Сірководень</option>
    </select>
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
