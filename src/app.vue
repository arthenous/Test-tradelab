<template>
    <div id="weatherBlock" class="row no-gutter">
        <div class="col-xs-12">
            <label>Sélectionner votre ville</label>
            <select class="col-xs-12" v-model="selectedCity" @change="getMeteo()">
                <option v-for="(city, key) in cities" :value="key">{{ city.nm }}</option>
            </select>
            <div class="col-xs-12">
                <div class="current-day-meteo" v-if="currentDayMeteo" >
                    <div class="text-center title city">{{ city.nm }}</div>
                    <div class="col-xs-12 text-center weather" :class="todayWeather"></div>
                    <div class="text-center temp"> {{ todayTemperature }}</div>
                </div>
                <div v-else>
                    <div class="lds-ring"><div></div><div></div><div></div><div></div></div>
                </div>

                <next-day
                    :minTemperature="typeof nextDayMeteo[n] !== 'undefined' ? Math.floor(nextDayMeteo[n].main.temp_min) : 0"
                    :maxTemperature="typeof nextDayMeteo[n] !== 'undefined' ? Math.floor(nextDayMeteo[n].main.temp_max)  : 0"
                    :idWeather="typeof nextDayMeteo[n] !== 'undefined' ? nextDayMeteo[n].weather[0].id : 0"
                    :loading="nextDayMeteo.length < 2"
                    :nb="nextDayMeteo.length"
                    :time="typeof nextDayMeteo[n] !== 'undefined' ? nextDayMeteo[n].dt_txt : ''"
                    v-for="n in [0,1,2]"
                >{{n}}
                </next-day>
            </div>
        </div>
    </div>
</template>

<script>
    import cities from "./cities-fr.json";
    import nextDay from "./nextDay.vue";

    export default {
        components: {
            nextDay
        },
        data () {
            return {
                cities,
                selectedCity: 0,
                currentDayMeteo: null,
                nextDayMeteo: []
            }
        },
        computed: {
            city: function() {
                if (
                    typeof this.cities === 'undefined'
                    || typeof this.cities[this.selectedCity] === 'undefined'
                ) {
                    return {};
                }
                return this.cities[this.selectedCity];
            },
            todayWeather: function () {
                return 'wi wi-icon-' + this.currentDayMeteo.weather[0].id;
            },
            todayTemperature: function() {
                return Math.floor(this.currentDayMeteo.main.temp) + '°';
            }
        },
        methods: {
            getMeteo: function() {
                let _this = this;
                // On remet la variable à null pour pouvoir afficher
                // le "loader" le temps que l'api réponde.
                this.currentDayMeteo = null;

                // On va appeler l'api openceathermap pour récupérer la météo du jour.
                $.ajax({
                    url: '//api.openweathermap.org/data/2.5/weather?appid=0039723af85bfb77a34e42c0fbea5b5e&units=metric&id=' + this.city.id,
                    dataType: 'json',
                    success: (json) => {
                        _this.currentDayMeteo = json;
                    }
                });

                // On ré-initialise le tableau contenant les
                // prévisions méteo des 3 prochain jours.nextDayMeteo
                // Ceci permet aussi d'afficher le "loader" pour le composant nextDay
                this.nextDayMeteo = [];

                // On va appeler l'api openceathermap pour récupérer les prévisons
                // météo pour les prochains jours.
                let currentDate = new Date();
                $.ajax({
                    url: '//api.openweathermap.org/data/2.5/forecast?appid=0039723af85bfb77a34e42c0fbea5b5e&units=metric&id=' + this.city.id,
                    dataType: 'json',
                    success: (json) => {
                        json.list.forEach(function(data) {
                            let date = new Date(data.dt_txt);

                            // L'api renvoie une liste de prévision méteo dans un ordres chronologique avec un intervalle de 3 heures.
                            // Donc si la date change, c'est qu'on est au lendemain.
                            // On prendra la première prévision à partir de 12h00.
                            if (
                                currentDate.getDate() != date.getDate()
                                && date.getHours() >= 12
                                && _this.nextDayMeteo.length < 3
                            ) {
                                _this.nextDayMeteo.push(data);
                                currentDate = date;
                            }

                        });
                    }
                });
            }
        },
        mounted: function () {
            this.$nextTick(function () {
                // Le code de cette fonction ne sera lancé
                // que la vue sera totallement chargée.
                // On pourra donc lancer la fonction getMeteo à
                // ce moment là.
                this.getMeteo();
            })
        }
    }
</script>

<style type="text/css" lang="scss">
    $bg-color: #9575CD;
    $title-color: #7E57C2;

    #weatherBlock {
        background: $bg-color;
        color: white;

        label {
            display: inline-block;
        }

        .title {
            background: $title-color;
        }

        select {
            color: black;
            margin-bottom: 10px;
        }

        .current-day-meteo {
            div {
                margin-bottom: 5px;
            }
            .city {
                text-transform: capitalize;
            }
            .temp {
                font-size: 20px;
                font-weight: bold;
            }

            .weather {
                font-size: 50px;
            }
        }
    }

    .lds-ring {
        display: inline-block;
        position: relative;
        width: 64px;
        height: 64px;

        div {
            box-sizing: border-box;
            display: block;
            position: absolute;
            width: 51px;
            height: 51px;
            margin: 6px;
            border: 6px solid #fff;
            border-radius: 50%;
            animation: lds-ring 1.2s cubic-bezier(0.5, 0, 0.5, 1) infinite;
            border-color: #fff transparent transparent transparent;
        }
        div:nth-child(1) {
            animation-delay: -0.45s;
        }
        div:nth-child(2) {
            animation-delay: -0.3s;
        }
        div:nth-child(3) {
            animation-delay: -0.15s;
        }
    }
    @keyframes lds-ring {
        0% {
            transform: rotate(0deg);
        }
        100% {
            transform: rotate(360deg);
        }
    }
</style>