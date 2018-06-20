<template>
        <div class="col-xs-4 no-padding text-center">
            <div class="title">{{ dayDate }}</div>
            <div v-if="!loading">
                <div :class="weather"></div>
                <div> {{ maxTemperature }}°</div>
                <div> {{ minTemperature }}°</div>
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
            dayDate: {
                type: String,
                required: true
            }
        },
        computed: {
            weather: function() {
                return 'wi wi-icon-' + this.idWeather;
            }
        }
    }
</script>

<style type="text/css">
    .no-padding {
        padding: 0;
    }

</style>