<template>
    <div id="weatherBlock" class="row no-gutter">
        <div class="col-xs-12">
            <label>Sélectionner votre ville</label>
            <select class="col-xs-12" v-model="selectedCity" @change="getMeteo()">
                <option v-for="(city, key) in cities" :value="key">{{ city.nm }}</option>
            </select>
            <div class="col-xs-12 current-day-meteo">
                    <div class="text-center title city">{{ city.nm }}</div>
                    <div v-if="currentDayMeteo" >
                        <div class="col-xs-12 text-center weather" :class="todayWeather"></div>
                        <div class="text-center temp"> {{ todayTemperature }}</div>
                    </div>
                    <div class="text-center" v-else>
                        <div class="lds-ring"><div></div><div></div><div></div><div></div></div>
                    </div>

                <next-day
                    :minTemperature="typeof nextDayMeteo[n] !== 'undefined' ? Math.floor(nextDayMeteo[n].min_temp) : 0"
                    :maxTemperature="typeof nextDayMeteo[n] !== 'undefined' ? Math.floor(nextDayMeteo[n].max_temp)  : 0"
                    :idWeather="typeof nextDayMeteo[n] !== 'undefined' ? nextDayMeteo[n].weather : 0"
                    :loading="nextDayMeteo.length < 2"
                    :dayDate="typeof nextDayMeteo[n] !== 'undefined' ? nextDayMeteo[n].dayDate : (new Date(86400000 * n)).toDateString().substr(0,3)"
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
                    url: 'http://api.openweathermap.org/data/2.5/weather?appid=0039723af85bfb77a34e42c0fbea5b5e&units=metric&id=' + this.city.id,
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
                $.ajax({
                    url: 'http://api.openweathermap.org/data/2.5/forecast?appid=0039723af85bfb77a34e42c0fbea5b5e&units=metric&id=' + this.city.id,
                    dataType: 'json',
                    success: (json) => {
                        // Stocke la date actuelle
                        let currentDate = new Date();

                        // Stocke la date du précedent jour parcouru
                        let dateN1 = new Date();

                        // Stocke la température min pour la journée
                        let min_temp = 0;

                        // Stocke la température max pour la journée
                        let max_temp = 0;

                        // Stocke l'enssemble des weather de la journée
                        // on selectionnera ensuite le weather le plus représenté
                        // dans le tableau pour afficher le temps de la journée.
                        let weather  = [];

                        // Fonction qui s'occupera d'extraire le weather le plus représenté dans le tableau.
                        function averageWeather(arr){
                            return arr.sort((a,b) =>
                                  arr.filter(v => v===a).length
                                - arr.filter(v => v===b).length
                            ).pop();
                        }

                        json.list.forEach(function(data) {
                            let parsingDate = new Date(data.dt_txt);

                            // On a changer de date, il faut donc envoyer les températures min et max
                            // et ensuite le ré-initialisé.
                            if (
                                dateN1.getDate() !== parsingDate.getDate()
                                && _this.nextDayMeteo.length < 3
                            ) {
                                //On sauvegarde les donnée pour la journée.
                                if (dateN1.getDate() !== currentDate.getDate()) {
                                    _this.nextDayMeteo.push({
                                        min_temp,
                                        max_temp,
                                        'weather': averageWeather(weather),
                                        'dayDate': dateN1.toDateString().substr(0,3)
                                    });
                                }

                                //On ré-initalise les variables
                                dateN1 = parsingDate;
                                min_temp = data.main.temp;
                                max_temp = data.main.temp;
                                weather = [];
                                weather.push(data.weather[0].id);
                            } else {
                                if (min_temp > data.main.temp) {
                                    min_temp = data.main.temp;
                                }
                                if (max_temp < data.main.temp) {
                                    max_temp = data.main.temp;
                                }
                                weather.push(data.weather[0].id);
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
        padding-top: 10px;
        border-radius: 3px;

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