<script setup lang="ts">
import axios from 'axios'
import { ref, watch } from 'vue'

const weatherIcon = 'http://openweathermap.org/img/wn/'

const props = defineProps<{
    location: string
    tempFormat: string // metric | imperial
    timeFormat: string
    aviation?: string // boolean
}>()

function calculateDewPoint(temp: number, humidity: number) {
    const a = 17.27
    const b = 237.7
    const gamma = (a * temp) / (b + temp) + Math.log(humidity / 100)
    const dewPoint = (b * gamma) / (a - gamma)
    return dewPoint
}

function getCloudCoverageCode(cloudiness: number) {
    switch (true) {
        case cloudiness === 0:
            return 'SKC' // Sky clear
        case cloudiness <= 25:
            return 'FEW' // Few clouds
        case cloudiness <= 50:
            return 'SCT' // Scattered clouds
        case cloudiness <= 87:
            return 'BKN' // Broken clouds
        case cloudiness <= 100:
            return 'OVC' // Overcast
        default:
            return 'Invalid cloudiness percentage' // Error handling
    }
}

let cld = ref('')
let weather: any = ref({})

function getWeather() {
    try {
        axios
            .get(
                `https://api.openweathermap.org/data/2.5/weather?q=${props.location}&units=${props.tempFormat}&appid=450da22d8cd2d079e4fc50144d7b15ef`,
            )
            .then((response) => {
                if (response.status !== 200) {
                    console.error('Invalid response')
                    return
                }

                weather.value = {
                    ...response.data,
                    sunrise: new Date(response.data.sys.sunrise * 1000),
                    sunset: new Date(response.data.sys.sunset * 1000),
                }

                cld = weather?.clouds?.all
                    ? getCloudCoverageCode(weather?.clouds?.all)
                    : ('SKC' as any)
            })
            .catch((err) => {
                console.error(err)
            })
    } catch (err) {
        console.error(err)
    }
}

getWeather()

watch(
    () => props,
    () => {
        getWeather()

        setInterval(() => {
            getWeather()
        }, 10 * 60 * 1000)
    },
    { immediate: true },
)
</script>

<template>
    <!-- <Weather location="Póvoa de Varzim" timeFormat="24H" tempFormat="metric"/> -->
    <!-- REMOVE THE MAX WIDTH and MT IN PRODUCTION -->
    <div
        class="p-2 pr-4 ring-1 ring-neutral-300/50 dark:ring-neutral-700/50 rounded-md grid grid-cols-4"
    >
        <div class="col-span-1 justify-center flex">
            <img
                :src="`${weatherIcon}${weather?.weather?.[0]?.icon}@2x.png`"
                v-if="weather?.weather?.[0].icon"
                class="max-h-18"
            />
        </div>
        <div class="col-span-2 grid grid-cols-2">
            <div class="col-span-2">
                <h1 class="text-xl font-bold col-span-2">
                    {{ weather?.name }}
                </h1>
                <div class="font-bold col-span-2">
                    {{ Math.round(weather?.main?.temp) }}°{{
                        props.tempFormat == 'metric' ? 'C' : 'F'
                    }}
                </div>
                <div class="grid grid-cols-3 text-sm opacity-75">
                    <div class="col-span-2">
                        {{
                            weather?.weather?.[0]?.description.replace(
                                /\b\w/g,
                                (char: string) => char.toUpperCase(),
                            )
                        }}
                    </div>
                    <div class="text-blue-500 font-semibold">
                        {{
                            weather?.rain?.['1h']
                                ? `${weather?.rain?.['1h']}mm/hr`
                                : ''
                        }}
                    </div>
                </div>
            </div>
        </div>

        <div class="cf-mono opacity-75 text-right">
            <div title="Sunrise">
                🌅
                {{
                    new Date(weather?.sys?.sunrise * 1000)
                        .toLocaleTimeString(
                            timeFormat === '12H' ? 'en-US' : 'en-GB',
                            { hour: '2-digit', minute: '2-digit' },
                        )
                        .toLowerCase()
                        .replace(' ', '')
                }}
            </div>
            <div title="Sunset">
                🌇
                {{
                    new Date(weather?.sys?.sunset * 1000)
                        .toLocaleTimeString(
                            timeFormat === '12H' ? 'en-US' : 'en-GB',
                            { hour: '2-digit', minute: '2-digit' },
                        )
                        .toLowerCase()
                        .replace(' ', '')
                }}
            </div>
        </div>

        <div
            class="col-span-3 mt-4 font-sm opacity-75 pt-1"
            v-if="props.aviation"
        >
            {{ weather?.wind?.deg }}°/{{
                Math.round(weather?.wind?.speed * 1.944)
            }}{{
                weather?.wind?.gust > 0
                    ? `G${Math.round(weather?.wind?.gust * 1.944)}`
                    : ''
            }}kt
            {{
                weather?.visibility >= 5000
                    ? `${weather?.visibility / 1000}km`
                    : weather?.visibility
            }}
            {{ cld }}
            {{ Math.round(weather?.main?.temp) }}/{{
                Math.round(
                    calculateDewPoint(
                        weather?.main?.temp,
                        weather?.main?.humidity,
                    ),
                )
            }}
            Q{{ weather?.main?.pressure }}
        </div>
        <div class="mt-4" v-if="props.aviation">
            <span
                class="ring-1 ring-green-700 bg-green-700/25 text-white rounded-full px-4 py-1 text-xs"
            >
                VFR
            </span>
        </div>
    </div>
</template>
