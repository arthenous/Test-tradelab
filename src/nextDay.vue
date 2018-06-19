<template>
        <div class="col-xs-4 no-padding text-center">
            <div class="title">{{ day }}</div>
            <div v-if="!loading">
                <div :class="weather"></div>
                <div> {{ minTemperature }}°</div>
                <div> {{ maxTemperature }}°</div>
            </div>
            <div v-else>
                <div class="lds-ring"><div></div><div></div><div></div><div></div></div>
            </div>
        </div>
</template>

<script>
    export default {
        data () {
            return {
            }
        },
        props: {
            nb: {
                type: Number
            },
            minTemperature: {
                type: Number,
                required: true
            },
            maxTemperature: {
                type: Number,
                required: true
            },
            idWeather: {
                type: Number,
                required: true
            },
            loading: {
                type: Boolean,
                required: true
            },
            time: {
                type: String,
                required: true
            }
        },
        computed: {
            weather: function() {
                return 'wi wi-icon-' + this.idWeather;
            },
            day: function() {
                // La fonction renvoie juste les 3 premières
                // lettre du jour.
                let date = new Date(this.time);

                return date.toDateString().substr(0,3);
            }
        }
    }
</script>

<style type="text/css">
    .no-padding {
        padding: 0;
    }

</style>