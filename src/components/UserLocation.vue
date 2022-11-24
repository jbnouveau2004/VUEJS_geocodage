<template>
    <div>
        <section class="ui two column centered grid" style="position:relative;z-index:1;">
            <div class="column">
                <form class="ui segment large form">
                    <div class="ui message red" v-show="error">{{error}}</div>  <!-- v-show="error" pour rendre visible et {{error}} pour afficher l'erreur -->
                    <div class="ui segment">
                        <div class="field">
                            <div class="ui right icon input large" :class="{loading:spinner}"> <!-- Ajout de la class "loading" si spinner retourne true -->
                                <input type="text" placeholder="Entrez votre adresse" v-model="address" id="autocomplete"> <!-- v-model="address" pour afficher l'erreur -->
                                <i class="dot circle link icon" @click="locatorButtonPressed"></i>
                            </div>
                        </div>
                        <button class="ui button">Afficher la carte</button>
                    </div>
                </form>
            </div>
        </section>
        <section id="map"></section>
    </div>
</template>

<script>

import axios from "axios";

export default {

    data() {
        return {
            address: "", // retourne la data dans address là où sera positionné "address" dans le code balisé
            error: "", // retourne les erreurs de la data là où sera positionné "error" dans le code balisé
            spinner: false
        }
    },

    mounted() {
        let autocomplete = new google.maps.places.Autocomplete(
            document.getElementById("autocomplete"),
            {
                bounds: new google.maps.LatLngBounds(
                    new google.maps.LatLng(47.8167, 6.3833) // autosuggestion à partir de Luxeuil-les-Bains
                )
            }
        );

        autocomplete.addListener("place_changed", () => {
            let place = autocomplete.getPlace();
            console.log(place);
            this.showUserLocationOnTheMap(place.geometry.location.lat(), place.geometry.location.lng())
        });
    },

    methods: {
        locatorButtonPressed(){

            this.spinner = true; // retourne spinner à true au démarrage de la fonction locatorButtonPressed 

            if(navigator.geolocation){
                navigator.geolocation.getCurrentPosition(
                position => {
                    this.getAddressFrom(position.coords.latitude, position.coords.longitude); // appel la fonction affichage de l'adresse avec l'altitude et la longitude du client
                    console.log(position.coords.latitude); // position du client
                    console.log(position.coords.longitude); // position du client

                    this.showUserLocationOnTheMap(position.coords.latitude, position.coords.longitude); // appel la fonction affichage de la map avec l'altitude et la longitude du client

                },
                error => {
                    this.error = "Nous n'avons pas l'autorisation de connaitre votre position, merci de rentrer votre adresse manuellement" // retourne l'erreur dans error
                    console.log(error.message);
                    this.spinner = false; // retourne spinner à false à la fin
                }
                );
            } else {
                this.error = "Votre navigateur ne supporte pas l'API de geolocalisation" // retourne l'erreur dans error
                console.log("Votre navigateur ne supporte pas l'API de geolocalisation");
                this.spinner = false; // retourne spinner à false à la fin
            }
        },
        getAddressFrom(lat, long){
            axios.get("https://maps.googleapis.com/maps/api/geocode/json?latlng=" + lat + "," + long + "&key=YOURKEY")
            .then(response => {
                if(response.data.error_message) {
                    this.error = response.data.error_message // retourne l'erreur dans error
                    console.log(response.data.error_message); // erreur retourné par le serveur
                } else {
                    this.address = response.data.results[0].formatted_address // retourne le résultat dans data
                    console.log(response.data.results[0].formatted_address); // retourne le résultat dans la console
                }
                this.spinner = false; // retourne spinner à false à la fin
            })
            .catch(error => {
                this.error = error.message // retourne l'erreur dans error
                console.log(error.message) // erreur retourné si GET est mal envoyé
                this.spinner = false; // retourne spinner à false à la fin
            })
        },
        showUserLocationOnTheMap(latitude, longitude){
            // affichage de la map
            let map = new google.maps.Map(document.getElementById("map"), {
                zoom:15,
                center: new google.maps.LatLng(latitude, longitude),
                mapTypeId: google.maps.MapTypeId.ROADMAP
            })
            // affichage de la position dans la connaitre
            new google.maps.Marker({
                position: new google.maps.LatLng(latitude, longitude),
                map: map
            })
        }
    }
}
</script>

<style>
.ui.button,
.dot.circle.icon {
    background-color: blue;
    color: white;
}

.pac-icon {
    display: none;
}

.pac-item {
    padding: 10px;
    font-size: 16px;
    cursor: pointer;
}

.pac-item:hover {
    background-color: #ececec;
}

.pac-item-query {
    font-size: 16px;
}

#map {
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    background: grey;
}

</style>