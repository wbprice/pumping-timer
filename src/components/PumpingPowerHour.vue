<script setup>
import { ref } from 'vue';

const PUMP = "PUMP";
const REST = "REST";
const DONE = "DONE";

const DEBUG = false;
const TWENTY_MINS_IN_MS = DEBUG ? 20 * 600 : 20 * 60000;
const TEN_MINS_IN_MS = DEBUG ? 10 * 600 : 10 * 60000;
const FIVE_MINS_IN_MS = DEBUG ? 5 * 600 : 5 * 60000;

const audio = ref(HTMLAudioElement);

// Scheduling internals
let active = ref(false);
let scheduleIndex = 0;
const schedule = [
    { activity: PUMP, duration: TWENTY_MINS_IN_MS },
    { activity: REST, duration: TEN_MINS_IN_MS },
    { activity: PUMP, duration: TEN_MINS_IN_MS },
    { activity: REST, duration: FIVE_MINS_IN_MS },
    { activity: PUMP, duration: FIVE_MINS_IN_MS },
    { activity: REST, duration: FIVE_MINS_IN_MS },
    { activity: PUMP, duration: FIVE_MINS_IN_MS },
];

let currentSchedule = schedule[scheduleIndex];

// The labels to be displayed to the user
let duration = currentSchedule.duration;
let timeLabel = ref(millisToMinutesAndSeconds(duration));
let activityLabel = ref(currentSchedule.activity);

// Convert milleseconds to a minute and seconds string
function millisToMinutesAndSeconds(millis) {
    var minutes = Math.floor(millis / 60000);
    var seconds = ((millis % 60000) / 1000).toFixed(0);
    return (
        seconds == 60 ?
            (minutes + 1) + ":00" :
            minutes + ":" + (seconds < 10 ? "0" : "") + seconds
    );
}

// timer functionality
function timerLoop() {
    if (active.value) {
        duration -= 1000;
        timeLabel.value = millisToMinutesAndSeconds(duration);

        if (duration <= 0) {
            // chime bell effect
            audio.value.play();
            // Is there another scheduled activity up next?
            if (scheduleIndex < schedule.length - 1) {
                scheduleIndex += 1;
                currentSchedule = schedule[scheduleIndex];
                duration = currentSchedule.duration
                activityLabel.value = currentSchedule.activity;
            } else {
                // If not, we're done
                active = false;
                activityLabel.value = DONE;
            }
        }
    }
}

// invoke timer loop once a second
setInterval(timerLoop, 1000);
</script>

<template>
    <h1>Pumping Power Hour</h1>
    <section class="container">
        <div class="timer" :class="{ active: activityLabel === PUMP }">
            <h1>{{ activityLabel }}</h1>
            <h2>{{ timeLabel }}</h2>
            <audio hidden="true" ref="audio">
                <source src="./../assets/alarmBell.mp3" type="audio/mpeg">
            </audio>
        </div>
    </section>

    <section class="interface">
        <button @click="active = !active">
            {{ active ? "PAUSE" : "PLAY" }}
        </button>
    </section>

    <p class="dedication">For Rosalina ❤️</p>
</template>

<style scoped>
.container {
    display: flex;
    justify-content: center;
    align-items: center;
}

.timer {
    background-color: red;
    border-radius: 100%;
    min-width: 300px;
    min-height: 300px;
    display: flex;
    align-content: center;
    justify-content: center;
    flex-direction: column;
}

.timer.active {
    background-color: green;
}

.timer h1 {
    font-size: 4em;
    margin: 0;
    color: white;
}

.timer h2 {
    font-size: 2em;
    color: white;
    margin: 0;
}

.interface button {
    font-size: 2em;
    margin-top: 1em;
}

.dedication {
    margin-top: 1em;
}

</style>
