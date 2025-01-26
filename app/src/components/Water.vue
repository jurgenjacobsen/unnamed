<script setup lang="ts">
import { computed, ref, Ref } from 'vue'

let props = defineProps<{
    goal?: string
    progress?: string
    format?: string // 'ml' | 'l'
}>()

// Local reactive progress to update dynamically
const localProgress = ref(Number(props.progress ?? 0))
let goal = Number(props.goal) >= 0 && props.goal ? Number(props.goal) : 3000

const totalBlocks = computed(() => {
    if (isNaN(goal) || goal <= 0) {
        console.error('Invalid goal')
        return 3
    }
    return goal / 1000
})

const progress = computed(() => {
    let goalValue = goal
    let progressValue = localProgress.value

    const percentage = (progressValue / goalValue) * 100
    let filledBlocks = Math.min(
        totalBlocks.value,
        Math.max(0, Math.floor((percentage / 100) * totalBlocks.value)),
    )

    if (props.format && props.format === 'l') {
        goalValue = goalValue / 1000
        progressValue = progressValue / 1000
    } else if (props.format && props.format !== 'ml' && goal >= 1000) {
        goalValue = goalValue / 1000
        progressValue = progressValue / 1000
    }

    if (goalValue < progressValue) {
        filledBlocks = totalBlocks.value
    }

    return {
        filledBlocks,
        percentage,
        goal: goalValue,
        progress: progressValue,
    }
})

const drinkVolume: Ref<string> = ref('')

function drinkClick() {
    const volume = Number(drinkVolume.value)
    if (isNaN(volume) || volume <= 0) {
        console.error('Invalid drink volume')
        drinkVolume.value = ''
        return
    }

    // Add the drink volume to localProgress
    localProgress.value += volume

    // Clear input field
    drinkVolume.value = ''
}
</script>

<template>
    <!-- <Water goal="" progress="2500" format="ml"/> -->
    <!-- IN PRODUCTION REMOVE MAX WIDTH -->
    <div
        class="p-2 ring-1 ring-neutral-300/50 dark:ring-neutral-700/50 rounded-md"
    >
        <div class="flex justify-between relative">
            <span>
                {{
                    props?.format == 'l'
                        ? progress.progress?.toFixed(1).replace('.0', '')
                        : progress.progress
                }}
                <span class="opacity-50">{{
                    props?.format == 'l' ? 'L' : 'ml'
                }}</span>
            </span>
            <span
                v-if="progress.percentage >= 100"
                class="text-blue-600/75 cf-mono text-xs font-bold absolute left-1/2 transform -translate-x-1/2"
            >
                Daily Goal Reached
            </span>
            <span>
                <span class="opacity-75">Goal</span> {{ progress.goal
                }}<span class="opacity-50">{{
                    props?.format == 'l' ? 'L' : 'ml'
                }}</span>
            </span>
        </div>

        <div
            :style="{
                gridTemplateColumns: `repeat(${totalBlocks}, minmax(0, 1fr))`,
            }"
            class="grid gap-1 rounded-md p-1 mt-1 ring-1 ring-neutral-300/50 dark:ring-neutral-700/50"
        >
            <div
                v-for="block in totalBlocks"
                class="min-h-4 rounded-md duration-200"
                :class="[
                    progress.filledBlocks >= block
                        ? 'bg-blue-700'
                        : 'bg-neutral-700',
                    { 'hover:bg-blue-600/50': progress.filledBlocks < block },
                ]"
            ></div>
        </div>

        <div class="mt-2 flex justify-between">
            <div
                class="min-w-1/4 max-w-2/5 ring-1 ring-neutral-300/50 dark:ring-neutral-700/50 p-1 rounded-md"
            >
                <input
                    type="number"
                    name="drink-volume"
                    id="drink-volume"
                    v-model="drinkVolume"
                    class="max-w-3/4 px-2 focus:outline-none text-neutral-900 dark:text-white/75"
                />
                <span class="pl-2 opacity-50">{{
                    props?.format == 'l' ? 'L' : 'ml'
                }}</span>
            </div>

            <button
                class="w-1/4 bg-blue-700 rounded-md p-1 cursor-pointer hover:opacity-75 duration-150"
                id="drink"
                @click="drinkClick"
            >
                Drink
            </button>
        </div>
    </div>
</template>

<style scoped>
input[type='number']::-webkit-outer-spin-button,
input[type='number']::-webkit-inner-spin-button {
    -webkit-appearance: none;
    margin: 0;
}

input[type='number'] {
    -moz-appearance: textfield;
}
</style>
