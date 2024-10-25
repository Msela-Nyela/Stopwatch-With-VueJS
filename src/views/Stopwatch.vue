<template>
    <div class="stopwatch-container">
        <svg class="stopwatch-circle" viewBox="0 0 100 100">
            <circle cx="50" cy="50" r="48" stroke="#333" stroke-width="4" />
            <circle cx="50" cy="50" r="48" stroke="#00ff00" stroke-width="4" stroke-dasharray="301.59"
                :stroke-dashoffset="progressCircleOffset" id="progress-circle" />
        </svg>
        <div class="stopwatch-display">{{ formattedTime }}</div>
        <div class="controls">
            <button id="reset" class="btn" @click="resetOrLap">{{ resetLapText }}</button>
            <button id="startStop" :class="{ 'stop': isRunning }" class="btn" @click="startStop">{{ startStopText
                }}</button>
        </div>
        <div class="laps">
            <div v-for="(lap, index) in laps" :key="index" class="lap">
                <span>Lap {{ laps.length - index }}</span>
                <span>{{ formatTime(lap) }}</span>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue';

const isRunning = ref(false);
const startTime = ref(null);
const elapsedTime = ref(0);
const laps = ref([]);
const timer = ref(null);

const updateDisplay = () => {
    elapsedTime.value = Date.now() - startTime.value;
    progressCircleOffset.value = 301.59 - ((elapsedTime.value / 1000) % 60) * (301.59 / 60);
};

const startStop = () => {
    if (!isRunning.value) {
        isRunning.value = true;
        startTime.value = Date.now() - elapsedTime.value;
        timer.value = setInterval(updateDisplay, 10); 
    } else {
        isRunning.value = false;
        clearInterval(timer.value);
    }
};

const resetOrLap = () => {
    if (isRunning.value) {
        laps.value.unshift(elapsedTime.value);
    } else {
        clearInterval(timer.value);
        isRunning.value = false;
        elapsedTime.value = 0;
        laps.value = [];
    }
};

const formatTime = (ms) => {
    const minutes = Math.floor(ms / 60000);
    const seconds = Math.floor((ms % 60000) / 1000);
    const centiseconds = Math.floor((ms % 1000) / 10);
    return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}.${centiseconds.toString().padStart(2, '0')}`;
};

const formattedTime = computed(() => formatTime(elapsedTime.value));
const progressCircleOffset = ref(301.59);

const startStopText = computed(() => (isRunning.value ? 'Stop' : 'Start'));
const resetLapText = computed(() => (isRunning.value ? 'Lap' : 'Reset'));

onMounted(() => {
    if (isRunning.value) {
        timer.value = setInterval(updateDisplay, 10);
    }
});

onUnmounted(() => {
    clearInterval(timer.value);
});
</script>