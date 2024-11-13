<template>
    <div class="sidebar-map">
        <h2>Informações do Mapa</h2>
        <p>Latitude: {{ latitude }}</p>
        <p>Longitude: {{ longitude }}</p>
        <div v-if="locationInfo">
            <h3>Detalhes do Local:</h3>
            <p><strong>País: </strong> {{ locationInfo.country }}</p>
            <p><strong>Estado:</strong> {{ locationInfo.state }}</p>
            <p><strong>Cidade:</strong> {{ locationInfo.city }}</p>
            <p><strong>Rua:</strong> {{ locationInfo.road }}</p>
            <p><strong>Nome completo:</strong> {{ locationInfo.fullName }}</p>
        </div>
        <button @click="getUserLocation" class="location-button">Marcar Minha Localização</button>
    </div>
</template>

<script>
export default {
    name: 'SideBarMap',
    props: {
        latitude: {
            type: Number,
            default: 0
        },
        longitude: {
            type: Number,
            default: 0
        }
    },
    data() {
        return {
            locationInfo: null
        }
    },
    methods: {
        getUserLocation() {
            if ("geolocation" in navigator) {
                navigator.geolocation.getCurrentPosition(
                    position => {
                        const { latitude, longitude } = position.coords;
                        this.$emit('marcarLocalizacao', [latitude, longitude, 18]); // Adicionamos o nível de zoom 18
                        this.getLocationName(latitude, longitude);
                    },
                    error => {
                        console.error("Erro ao obter localização:", error.message);
                        alert("Não foi possível obter sua localização. Por favor, verifique as permissões do seu navegador.");
                    }
                );
            } else {
                alert("Geolocalização não é suportada pelo seu navegador.");
            }
        },
        async getLocationName(lat, lon) {
            try {
                const response = await fetch(`https://nominatim.openstreetmap.org/reverse?format=json&lat=${lat}&lon=${lon}&zoom=18&addressdetails=1`);
                const data = await response.json();
                
                this.locationInfo = {
                    country: data.address.country || 'Não disponível',
                    state: data.address.state || 'Não disponível',
                    city: data.address.city || data.address.town || data.address.village || 'Não disponível',
                    road: data.address.road || 'Não disponível',
                    fullName: data.display_name || 'Nome completo não disponível'
                };
            } catch (error) {
                console.error('Erro ao obter nome do local:', error);
                this.locationInfo = null;
            }
        }
    }
}
</script>

<style scoped>
.sidebar-map {
    padding: 20px;
    background-color: #f0f0f0;
    border-left: 1px solid #ccc;
    width: 280px;
    height: 100%;
    overflow-y: auto;
    box-sizing: border-box;
}

.location-button {
    margin-top: 20px;
    padding: 10px;
    background-color: #4CAF50;
    color: white;
    border: none;
    cursor: pointer;
    border-radius: 4px;
    font-size: 14px;
    width: calc(100% - 40px);
    display: block;
    margin-left: auto;
    margin-right: auto;
}

.location-button:hover {
    background-color: #45a049;
}

h2, h3 {
    margin-top: 0;
    margin-bottom: 15px;
}

p {
    margin-bottom: 10px;
}

.sidebar-map {
    scrollbar-width: none;
    -ms-overflow-style: none;
}

.sidebar-map::-webkit-scrollbar {
    display: none;
}
</style>
