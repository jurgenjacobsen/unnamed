<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'
import beepSound from '/sounds/beep.mp3'

const props = defineProps<{
    pomodoroDuration?: string
    breakDuration?: string
    beepSound?: string
}>()

const pomodoroDuration = props.pomodoroDuration
    ? Number(props.pomodoroDuration) * 60
    : 25 * 60 // 25 minutes in seconds
const breakDuration = props.breakDuration
    ? Number(props.breakDuration) * 60
    : 5 * 60

const timer = ref(pomodoroDuration)
const isPomodoro = ref(true) // True for Pomodoro session, false for break
const isRunning = ref(false)
const isBreak = ref(false)
const pomodoroCount = ref(0)
const startTime = ref<number | null>(null)

const timerDisplay = computed(() => {
    const minutes = Math.floor(timer.value / 60)
    const seconds = timer.value % 60
    return `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`
})

let intervalId: any
let beep: any

function startTimer() {
    if (isRunning.value) return
    isRunning.value = true

    startTime.value = Date.now() - (isPomodoro.value ? pomodoroDuration - timer.value : breakDuration - timer.value) * 1000

    intervalId = setInterval(() => {
        const elapsed = Math.floor((Date.now() - (startTime.value || 0)) / 1000)
        timer.value = Math.max((isPomodoro.value ? pomodoroDuration : breakDuration) - elapsed, 0)

        if (timer.value === 0) toggleSession()
    }, 1000)
}

function stopTimer() {
    isRunning.value = false
    clearInterval(intervalId)
}

function resetTimer(byUserAction = false) {
    stopTimer()
    if (byUserAction) {
        isPomodoro.value = true
        isBreak.value = false
        timer.value = pomodoroDuration
        isRunning.value = false
    } else {
        timer.value = isPomodoro.value ? pomodoroDuration : breakDuration
    }
}

function toggleSession() {
    if (isPomodoro.value) pomodoroCount.value++

    isPomodoro.value = !isPomodoro.value
    isBreak.value = !isBreak.value
    timer.value = isPomodoro.value ? pomodoroDuration : breakDuration

    if (props.beepSound) {
        try {
            beep.play()
        } catch (error) {
            console.error('Beep error:', error)
        }
    }

    resetTimer()
}

onMounted(() => {
    timer.value = pomodoroDuration
    beep = new Audio(beepSound)
})

onUnmounted(() => {
    clearInterval(intervalId)
})
</script>

<template>
    <div
        class="p-4 ring-1 ring-neutral-300/50 dark:ring-neutral-700/50 rounded-md"
    >
        <div class="flex justify-between">
            <span class="font-semibold opacity-75">Pomodoro Session</span>
            <span class="opacity-25">{{
                pomodoroCount > 0 ? `${pomodoroCount} Completed` : ''
            }}</span>
        </div>
        <div class="grid grid-cols-3 mt-1">
            <div class="col-span-1 flex items-center">
                <span class="cf-mono font-semibold text-3xl">
                    {{ timerDisplay }}
                </span>
                <span
                    v-show="isBreak"
                    class="ml-3 p-0.5 px-2 bg-neutral-500/25 ring-1 ring-neutral-300/50 dark:ring-neutral-500/50 rounded-md"
                    >Break</span
                >
            </div>

            <div class="col-span-2 grid grid-cols-3 gap-2">
                <button
                    class="bg-blue-700 rounded-md px-4 py-1 cursor-pointer hover:opacity-75 duration-150"
                    @click="startTimer"
                    :disabled="isRunning"
                >
                    {{ isRunning ? 'Running' : 'Start' }}
                </button>
                <button
                    class="bg-red-700 rounded-md px-4 py-1 cursor-pointer hover:opacity-75 duration-150"
                    @click="stopTimer"
                    :disabled="!isRunning"
                >
                    Stop
                </button>
                <button
                    class="bg-neutral-500 rounded-md px-4 py-1 cursor-pointer hover:opacity-75 duration-150"
                    @click="resetTimer(true)"
                >
                    Reset
                </button>
            </div>
        </div>
    </div>
</template>

<style scoped>
button:disabled {
    background-color: gray;
    cursor: not-allowed;
}
</style>
