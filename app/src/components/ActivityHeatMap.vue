<script setup lang="ts">
    import { computed } from "vue";

    const year = new Date().getFullYear();
    const firstDayOffset = new Date(year, 0, 1).getDay();

    const dayLabels = ["Sun","Mon", "Tue", "Wed", "Thu", "Fri", "Sat"];
    const months = [
      "Jan", "Feb", "Mar", "Apr", "May", "Jun",
      "Jul", "Aug", "Sep", "Oct", "Nov", "Dec",
    ];

    const legends = ["#404040", "#9be9a8", "#40c463", "#30a14e", "#216e39"];
    const activityData = [
        { date: new Date("2025-01-01"), activities: 5 },
        { date: new Date("2025-01-02"), activities: 3 },
        { date: new Date("2026-05-02"), activities: 3 },
    ]

    let populate = true; // Set to false to disable random data
    if(populate) {
        for (let i = 0; i < 150; i++) {
        const date = new Date(year, 0, Math.random() * 365); // Random date in the year
        const activities = Math.floor(Math.random() * 21); // Random contributions between 0 and 20
        activityData.push({ date, activities });
        }
    }

    const daysInYear = Array.from({ length: 365 }, (_, i) => {
        const date = new Date(year, 0, i + 1);
        const isoDate = date.toISOString().split("T")[0];
        return { date: isoDate, day: date.getDay(), activities: 0 };
    });

    activityData.forEach((entry) => {
        const day = daysInYear.find((d) => d.date === entry.date.toISOString().split("T")[0]);
        if (day) day.activities = entry.activities;
    });

    const activityGrid = computed(() => {
        const gridData = [];
        for (let i = 0; i < daysInYear.length; i++) {
            gridData.push(daysInYear[i]);
        }
        return gridData;
    });

    const grid = computed(() => {
        const startDays: string[] = [months[0]];
        let currentMonth = 0;

        daysInYear.forEach((day) => {
            const month = new Date(day.date).getMonth();
            if (month !== currentMonth) {
                startDays.push(months[month]);
                currentMonth = month;
            }
        });

        return startDays;
    })

    function getColor(activities: number) {
        if (activities === 0) return legends[0];
        if (activities < 5) return legends[1];
        if (activities < 10) return legends[2];
        if (activities < 20) return legends[3];
        return legends[4];
    }
</script>

<template>
    <div class="p-4 ring-1 ring-neutral-300/50 dark:ring-neutral-700/50 rounded-md">
        <h1>2025</h1>

        <div class="grid grid-cols-32">
            <div class="col-span-1">
              <!--flex flex-col justify-between h-28 mt-4-->
                <div class="grid grid-cols-1 grid-rows-7 mt-4">
                    <div v-for="day in dayLabels" :key="day" class="text-xs opacity-50">{{ day }}</div>
                </div>
            </div>

            <div class="col-span-31">
                <div class="h-4 w-full flex flex-row justify-between">
                    <div class="h-4 w-full flex flex-row justify-between">
                        <div
                        v-for="(month, index) in grid"
                        :key="index"
                        class="text-xs opacity-50"
                        :style="{ gridColumnStart: index + 1 }"
                        >
                        {{ month }}
                        </div>
                    </div>
                </div>

                <div class="grid grid-cols-53 grid-rows-7 gap-1">
                  <div
                      v-for="(day, index) in activityGrid"
                      :key="index"
                      class="w-3 h-3 rounded-xs"
                      :title="`${day.date}: ${day.activities} activities`"
                      :style="{ 
                        backgroundColor: getColor(day.activities), 
                        gridColumn: Math.floor((index + firstDayOffset) / 7) + 1, 
                        gridRow: ((index + firstDayOffset) % 7) + 1 
                      }">
                  </div>
                </div>

                <div>
                    <div class="flex items-center gap-1 mt-2">
                        <span>Less</span>
                        <div class="h-3 w-3 rounded-xs" v-for="(color, index) in legends" :key="index"
                            :style="{ backgroundColor: color }"></div>
                        <span>More</span>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
