<template>
    <div>
        <div class="route-input">
            <form>
                <label>
                    <!-- Départ : -->
                    <input v-model="from" placeholder="Ex: Paris, France" @input="searchFromOptions">
                    <div v-if="fromOptions.length">
                        <ul>
                            <li v-for="option in fromOptions" :key="option.place_id" @click="selectFromOption(option)">
                                {{ option.display_name }}
                            </li>
                        </ul>
                    </div>
                </label>
                <label>
                    <!-- Arrivée : -->
                    <input v-model="to" placeholder="Ex: Louviers, France" @input="searchToOptions">
                    <div v-if="toOptions.length">
                        <ul>
                            <li v-for="option in toOptions" :key="option.place_id" @click="selectToOption(option)">
                                {{ option.display_name }}
                            </li>
                        </ul>
                    </div>
                </label>
            </form>
        </div>

    </div>
</template>

<script>
import axios from 'axios';

export default {
    data() {
        return {
            from: '',
            to: '',
            fromOptions: [],
            toOptions: [],
            route: [
            ]
        };
    },
    methods: {
        async searchFromOptions() {
            if (this.from.length < 3) {
                this.fromOptions = [];
                return;
            }
            try {
                const response = await axios.get(`https://nominatim.openstreetmap.org/search?format=json&q=${this.from}`);
                this.fromOptions = response.data;
            } catch (error) {
                console.error("Erreur lors de la récupération des suggestions pour le départ :", error);
            }
        },
        async searchToOptions() {
            if (this.to.length < 3) {
                this.toOptions = [];
                return;
            }
            try {
                const response = await axios.get(`https://nominatim.openstreetmap.org/search?format=json&q=${this.to}`);
                this.toOptions = response.data;
            } catch (error) {
                console.error("Erreur lors de la récupération des suggestions pour l'arrivée :", error);
            }
        },
        selectFromOption(option) {
            this.from = option.display_name;
            this.fromOptions = [];
            this.updateRoute();
        },
        selectToOption(option) {
            this.to = option.display_name;
            this.toOptions = [];
            this.updateRoute();
        },
        async updateRoute() {
            if (this.from && this.to) {
                try {
                    const fromCoords = await this.getCoordinates(this.from);
                    const toCoords = await this.getCoordinates(this.to);

                    this.route.coordinates=[
                        [fromCoords.lat, fromCoords.lng],
                        [toCoords.lat, toCoords.lng]
                    ]
        
                    console.log("valeur de l'itinéraire", this.route);
                    this.$emit('selectRoute', this.route);
                } catch (error) {
                    console.error("Erreur lors de la récupération de l'itinéraire :", error);
                }
            }
        },
        async getCoordinates(location) {
            try {
                const response = await axios.get(`https://nominatim.openstreetmap.org/search?format=json&q=${location}`);
                if (response.data && response.data.length > 0) {
                    const { lat, lon } = response.data[0];
                    return { lat: parseFloat(lat), lng: parseFloat(lon) };
                } else {
                    throw new Error('Lieu non trouvé');
                }
            } catch (error) {
                console.error("Erreur lors de la récupération des coordonnées :", error);
                throw error;
            }
        }
    }
};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap');

.custom-font {
    font-family: 'Roboto', sans-serif;
}

.leaflet-container {
    z-index: 1;
    height: 100vh;
    width: 100%;
}

.route-input {
    position: absolute;
    top: 10px;
    left: 10px;
    background: white;
    padding: 10px;
    border-radius: 5px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    z-index: 10;
}

.route-input label {
    display: block;
    margin-bottom: 5px;
}

.route-input input {
    width: 200px;
    padding: 5px;
    margin-bottom: 10px;
}
</style>