<script setup>
import { ref } from 'vue';

const PUMP = "PUMP";
const REST = "REST";
const DONE = "DONE";

const TWENTY_MINS_IN_MS = 20 * 60000;
const TEN_MINS_IN_MS = 10 * 60000;
const FIVE_MINS_IN_MS = 5 * 60000;

const audio = ref(HTMLAudioElement);
const schedule = [
    { activity: PUMP, duration: TWENTY_MINS_IN_MS },
    { activity: REST, duration: TEN_MINS_IN_MS },
    { activity: PUMP, duration: TEN_MINS_IN_MS },
    { activity: REST, duration: FIVE_MINS_IN_MS },
    { activity: PUMP, duration: FIVE_MINS_IN_MS },
    { activity: REST, duration: FIVE_MINS_IN_MS },
    { activity: PUMP, duration: FIVE_MINS_IN_MS },
];

let active = true;
let scheduleIndex = 0;
let currentSchedule = schedule[scheduleIndex];
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
    if (active) {
        duration -= 1000;
        timeLabel.value = millisToMinutesAndSeconds(duration);
        activityLabel.value = currentSchedule.activity;

        if (duration <= 0) {
            // chime bell effect
            audio.value.play();
            // Is there another scheduled activity up next?
            if (scheduleIndex < schedule.length - 1) {
                scheduleIndex += 1;
                currentSchedule = schedule[scheduleIndex];
                duration = currentSchedule.duration
                // If not, we're done
            } else {
                active = false;
                activityLabel.value = DONE;
            }
        }
    }
}

setInterval(timerLoop, 1000);
</script>

<template>
    <h1>Pumping Power Hour</h1>
    <h2>{{ activityLabel }}</h2>
    <h2>{{ timeLabel }}</h2>
    <audio hidden="true" ref="audio">
        <source src="./../assets/alarmBell.mp3" type="audio/mpeg">
    </audio>
</template>

<style scoped>
</style>
