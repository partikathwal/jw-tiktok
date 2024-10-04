<template>
    <div class="relative flex justify-center items-center w-full h-full" @click="togglePause()">
        <video ref="backdrop" class="object-cover absolute z-0 h-full" :src="props.video.src + '#t=' + startOffset" preload="auto" loop></video>
        <div class="absolute z-10 w-full h-full backdrop-blur-md"></div>
        <video ref="foreground" class=" object-cover absolute z-20 h-2/3" :src="props.video.src + '#t=' + startOffset" preload="auto" loop></video>
        <div v-if="paused" class="absolute z-30 w-full h-full p-4 flex justify-center items-center text-8xl">⏸</div>
        <div class="absolute z-30 w-full h-full p-4 flex flex-col justify-end">
            <a class="text-white font-bold w-2/3 py-4" :href="props.video.url">▶ {{ props.video.title }}</a>
            <input @click.stop ref="seeker" :class="{ 'invisible': !paused || !initialized}" type="range">
        </div>
    </div>
</template>

<script setup>
//https://shihn.ca/posts/2020/media-pool/

import VideoContainer from '../VideoContainer.vue';

import { onMounted, ref, watch } from 'vue';

const props = defineProps(['video']);

const backdrop = ref();
const foreground = ref();
const seeker = ref();

const paused = ref(true);

const startOffset = 2; // seconds;

let initialized = false;

function play() {
    paused.value = false;

    foreground.value.play();
    backdrop.value.play();
}

function stop() {
    backdrop.value.pause();
    foreground.value.pause();
    backdrop.value.currentTime = startOffset;
    foreground.value.currentTime = startOffset;
}

function togglePause() {
    seeker.value.value = foreground.value.currentTime;
    paused.value = !paused.value;
    backdrop.value.paused ? backdrop.value.play() : backdrop.value.pause();
    foreground.value.paused ? foreground.value.play() : foreground.value.pause();
}

defineExpose({ play, stop });



function initialize(){
    seeker.value.max = foreground.value.duration;
    seeker.value.value = foreground.value.currentTime;
    seeker.value.addEventListener("input", (e) => {
        foreground.value.currentTime = e.target.value;
        backdrop.value.currentTime = e.target.value;
    })

    initialized = true;
}

onMounted(() => {
    backdrop.value.volume = 0;
    foreground.value.addEventListener("canplay", initialize);
    // foreground.value.load();
    // backdrop.value.load();
})

</script>