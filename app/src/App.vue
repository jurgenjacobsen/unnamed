<script setup lang="ts">
import ActivityHeatMap from './components/ActivityHeatMap.vue'
import Goals from './components/Goals.vue'
import Pomodoro from './components/Pomodoro.vue'
import Water from './components/Water.vue'
import Weather from './components/Weather.vue'
</script>

<template>
    <div class="grid grid-cols-3 gap-4 mt-4">
        <div class="col-span-1">
            <Water goal="5000" progress="2500" format="ml" />

            <Weather
                class="mt-4"
                location="Porto, Portugal"
                timeFormat="24H"
                tempFormat="metric"
            />

            <Pomodoro
                class="mt-4"
                pomodoroDuration="25"
                breakDuration="5"
                beepSound="true"
            />
        </div>
        <div class="col-span-2">
            <ActivityHeatMap />
            <Goals class="mt-4" />
        </div>
    </div>
</template>
