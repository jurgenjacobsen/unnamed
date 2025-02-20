<script setup lang="ts">
const currentYear = new Date().getFullYear();
const currentQ = Math.floor((new Date().getMonth() + 3) / 3);

const legends = {
    qs: [1,2,3,4]
}

const goals = [

]

const years = [
    {
        year: 2024,
        goals: [
            { q: 1, objective: '1' },
            { q: 1, objective: '2' },
            { q: 1, objective: '3' },
            { q: 2, objective: '4' },
            { q: 3, objective: '4' },
            { q: 3, objective: '5' },
            { q: 2, objective: '3' },
            { q: 1, objective: '4' },
            { q: 4, objective: '4' },
            { q: 3, objective: '5' },
        ]
    },
    {
        year: 2025,
        goals: [
            { q: 2, objective: '6' },
            { q: 2, objective: '7' },
            { q: 1, objective: '7' },
        ]
    },
    {
        year: 2026,
        goals: [
            { q: 3, objective: '6' },
            { q: 2, objective: '7' },
            { q: 1, objective: '6' },
            { q: 2, objective: '7' },
            { q: 4, objective: '7' },
            { q: 4, objective: '7' },
        ]
    }
]

</script>

<template>
  <div class="p-4 ring-1 ring-neutral-300/50 dark:ring-neutral-700/50 rounded-md">
    <h1>Goals</h1>

    <div class="grid grid-cols-1 gap-y-2">
        <div class="grid grid-cols-9 font-semibold opacity-25 cf-mono">
            <div>Year</div>
            <div v-for="(q,i) in 4" :index="i" class="col-span-2">Q{{ q }}</div>
        </div>

        <div v-for="(y,i) in years" :key="i" class="grid grid-cols-9 gap-2 ">

            <div>{{ y.year }}</div>

            <div 
            v-for="(q,i) in 4" 
            :key="i" 
            class="col-span-2" 
            :class="q === currentQ && y.year === currentYear ? 'ring-2 ring-blue-700 rounded-md px-2' : ''"
            :title="q === currentQ && y.year === currentYear ? 'Current Quarter' : ''">

                <div v-for="(goal, x) in y.goals" :key="x">
                    <div v-if="goal.q === q" 
                    class="ring-1 ring-neutral-300/50 dark:ring-neutral-700/50 rounded-md p-2 text-sm my-2 w-full truncate"
                    :title="goal.objective"
                    >
                        {{ goal.objective }}
                    </div>
                </div>

            </div>

            <hr class="col-span-9 border-neutral-300/20 dark:border-neutral-700/20 border-2 rounded-2xl my-2" v-if="years.find((yr)=>yr.year === y.year+1)"/>

        </div>
    </div>
  </div>
</template>
