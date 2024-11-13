<template>
  <div class="home-map">
    <div class="map-container">
      <l-map :zoom="zoom" :center="center" @ready="onMapReady" @moveend="onMapMove">
        <l-tile-layer :url="url"></l-tile-layer>
        <l-marker v-if="userLocation" :lat-lng="userLocation">
          <l-popup>Minha localização</l-popup>
        </l-marker>
        <l-marker v-for="incident in incidents" :key="incident.id" :lat-lng="[incident.latitude, incident.longitude]">
          <l-popup>
            <strong>{{ incident.type }}</strong><br>
            {{ incident.location }}<br>
            Data: {{ formatDate(incident.date) }}
          </l-popup>
        </l-marker>
      </l-map>
    </div>
    <SideBarMap 
      :latitude="center[0]" 
      :longitude="center[1]"
      @marcarLocalizacao="marcarLocalizacaoUsuario"
    />
    <Incidents 
      :latitude="center[0]"
      :longitude="center[1]"
      @incidentsLoaded="updateIncidents"
    />
  </div>
</template>

<script>
import { LMap, LTileLayer, LMarker, LPopup } from '@vue-leaflet/vue-leaflet'
import "leaflet/dist/leaflet.css"
import SideBarMap from './SideBarMap.vue'
import Incidents from './Incidents.vue'

export default {
  name: 'HomeMap',
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LPopup,
    SideBarMap,
    Incidents
  },
  data() {
    return {
      zoom: 10,
      center: [-23.5505, -46.6333], // Inicialmente centralizado em São Paulo
      url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      userLocation: null,
      map: null,
      incidents: []
    }
  },
  methods: {
    onMapReady(mapObject) {
      this.map = mapObject;
    },
    onMapMove(e) {
      const center = e.target.getCenter();
      this.center = [center.lat, center.lng];
    },
    marcarLocalizacaoUsuario(coordenadas) {
      this.userLocation = [coordenadas[0], coordenadas[1]];
      this.center = this.userLocation;
      this.zoom = coordenadas[2] || 18;
      if (this.map) {
        this.map.setView(this.center, this.zoom);
      }
    },
    updateIncidents(incidentsData) {
      this.incidents = incidentsData;
    },
    formatDate(dateString) {
      return new Date(dateString).toLocaleDateString('pt-BR');
    }
  }
}
</script>

<style scoped>
.home-map {
  display: flex;
  height: 100vh;
  width: 100vw;
}
.map-container {
  flex: 1;
  height: 100%;
}
</style>
