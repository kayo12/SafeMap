<script>
import { LMap, LTileLayer, LMarker, LPopup } from '@vue-leaflet/vue-leaflet';
import L from 'leaflet';
import incidentesData from '@/mocks/incidents.json';

export default {
  name: 'IncidentsList',
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LPopup
  },
  data() {
    return {
      incidents: [],
      loading: false,
      url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
      userIcon: L.divIcon({
        className: 'user-marker',
        html: '<div style="background-color: red; width: 12px; height: 12px; border-radius: 50%; border: 2px solid white;"></div>',
        iconSize: [16, 16],
        iconAnchor: [8, 8],
      }),
      incidentIcons: {
        Alto: this.createColoredIcon('red'),
        Médio: this.createColoredIcon('orange'),
        Baixo: this.createColoredIcon('yellow'),
      },
      mapCenter: [-23.550520, -46.633308], // Coordenadas de São Paulo
      zoom: 13, // Ajuste o zoom conforme necessário
      userLocation: null,
    };
  },
  methods: {
    createColoredIcon(color) {
      return L.divIcon({
        className: 'colored-marker',
        html: `<div style="background-color: ${color}; width: 20px; height: 20px; border-radius: 50%; border: 2px solid white;"></div>`,
        iconSize: [24, 24],
        iconAnchor: [12, 12],
      });
    },

    getIncidentIcon(gravidade) {
      const icon = this.incidentIcons[gravidade] || this.incidentIcons['Médio'];
      console.log(`Gravidade: ${gravidade}, Ícone: ${JSON.stringify(icon)}`);
      return icon;
    },

    async fetchIncidents() {
      this.loading = true;
      try {
        await new Promise(resolve => setTimeout(resolve, 500));
        
        this.incidents = incidentesData;
        

        this.$emit('incidentsLoaded', this.incidents);
      } catch (error) {
        console.error('Erro ao buscar incidentes:', error);
        this.incidents = [];
        this.$emit('incidentsLoaded', []);
      } finally {
        this.loading = false;
      }
    },

    calculateDistance(lat1, lon1, lat2, lon2) {
      // Implementação simplificada da fórmula de Haversine
      const R = 6371; // Raio da Terra em km
      const dLat = this.deg2rad(lat2 - lat1);
      const dLon = this.deg2rad(lon2 - lon1);
      const a =
        Math.sin(dLat/2) * Math.sin(dLat/2) +
        Math.cos(this.deg2rad(lat1)) * Math.cos(this.deg2rad(lat2)) *
        Math.sin(dLon/2) * Math.sin(dLon/2);
      const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
      return R * c;
    },

    deg2rad(deg) {
      return deg * (Math.PI/180);
    },

    updateMapSize() {
      if (this.$refs.map && this.$refs.map.mapObject) {
        this.$refs.map.mapObject.invalidateSize();
        // Ajusta o zoom para mostrar todos os marcadores e a localização do usuário
        const bounds = L.latLngBounds([...this.incidents.map(i => [i.latitude, i.longitude]), this.userLocation]);
        this.$refs.map.mapObject.fitBounds(bounds);
      }
    },

    onMapReady(mapObject) {
      console.log('Mapa pronto');
      this.mapObject = mapObject;
      this.updateMapSize();
    },

    getUserLocation() {
      if ("geolocation" in navigator) {
        navigator.geolocation.getCurrentPosition(position => {
          this.userLocation = [position.coords.latitude, position.coords.longitude];
          this.mapCenter = this.userLocation; // Centraliza o mapa na localização do usuário
          this.$nextTick(() => {
            this.updateMapSize();
          });
        }, error => {
          console.error("Erro ao obter localização:", error);
        });
      } else {
        console.log("Geolocalização não está disponível");
      }
    }
  },
  watch: {
    incidents: {
      handler() {
        this.$nextTick(() => {
          this.updateMapSize();
        });
      },
      deep: true
    }
  },
  mounted() {
    this.fetchIncidents();
    this.getUserLocation();
  }
}
</script>

<template>
  <div>
    <l-map ref="map" :zoom="zoom" :center="mapCenter">
      <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>
      <l-marker :lat-lng="mapCenter" :icon="userIcon">
        <l-popup>Você está aqui</l-popup>
      </l-marker>
      <l-marker 
        v-for="incident in incidents" 
        :key="incident.id"
        :lat-lng="[incident.latitude, incident.longitude]"
        :icon="getIncidentIcon(incident.gravidade)"
      >
        <l-popup>
          <h3>{{ incident.tipo }}</h3>
          <p>{{ incident.descricao }}</p>
          <p>Gravidade: {{ incident.gravidade }}</p>
        </l-popup>
      </l-marker>
    </l-map>
  </div>
</template>

<style scoped>
.user-marker, .colored-marker {
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
