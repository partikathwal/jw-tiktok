<template>
    <div ref="elementRef" class="bg-black h-dscreen relative snap-always snap-start" @click="togglePause()">
        <canvas ref="canvasRef"></canvas>
        <video class="hidden" ref="videoRef" :data-src="props.video.src" loop></video>
        <div v-if="controlsVisible" class="absolute top-0 z-30 w-full h-full p-4 flex justify-center items-center text-8xl">
            ⏸
        </div>
        <div v-if="isLoading" class="absolute top-0 z-40 w-full h-full flex justify-center items-center">
            <div class="animate-spin" style="width: 50px; height: 50px; border-left: 6px solid white; border-radius: 100%;"></div>
        </div>
        <div class="absolute top-0 z-50 w-full h-full p-4 flex flex-col justify-end items-start">
            <div class="w-full flex items-baseline justify-between">
                <div class="flex items-baseline gap-2 text-white font-bold">
                    <span>▶</span>
                    <a class="py-4 w-2/3" :href="props.video.url" target="_blank">{{ props.video.title }}</a>
                </div>
                <div @click.stop="toggleMute()" class="p-4 text-xl cursor-pointer">
                    {{ muted ? '🔇' :  '🔊' }}
                </div>
            </div>
            <input @click.stop ref="seekerRef" class="w-full" :class="{ 'invisible': !controlsVisible }" type="range">
        </div>
    </div>
</template>


<script setup>

import { onMounted, ref, watch, toRefs, computed } from 'vue';
import { state } from "./state.js";

const props = defineProps(['video']);

// workaround, since defineExpose stops me from accessing the html of this component
const elementRef = ref();
const getElementRef = computed(() => {
    return elementRef;
})

const canvasRef = ref();
const videoRef = ref();

const seekerRef = ref();
let seeker;

const controlsVisible = ref(false);

const isLoading = ref(false);

const { muted } = toRefs(state);

watch(muted, () => {
    video.muted = muted.value;
})

/** @type {HTMLCanvasElement} */
let canvas;

/** @type {HTMLVideoElement} */
let video;

// how much to scale down a full height video
const scale = 0.75;
const startOffset = 1.6;

onMounted(() => {
    // dom elements exist, store them
    canvas = canvasRef.value;
    video = videoRef.value;

    // initialize canvas properties
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;

    seeker = seekerRef.value;
    seeker.addEventListener("input", (e) => {
        video.currentTime = e.target.value;
        video.requestVideoFrameCallback((now, metadata) => {
            updateCanvas(now, metadata, true);
        });
    })

    video.muted = muted.value;
})

let videoFrameCallbackHandle;

function getShortTitle(){
    return props.video.title.split(":")[0];
}

async function play(fromBeginning){
    controlsVisible.value = false;
    if(fromBeginning){
        video.currentTime = startOffset;
    }
    load();
    const loadingTimeout = setTimeout(() => {
        isLoading.value = true;
    }, 500);
    await video.play();
    clearTimeout(loadingTimeout);
    isLoading.value = false;
    console.log("playing " + getShortTitle());
    videoFrameCallbackHandle = video.requestVideoFrameCallback(updateCanvas);
}

function pause(){
    seeker.value = video.currentTime;
    controlsVisible.value = true;
    video.pause();
    video.cancelVideoFrameCallback(videoFrameCallbackHandle);
}

function stop(){
    // called when switching away from the video
    // don't want pause icon and seeker to show
    // so controls shouldn't be seen when going back to it
    controlsVisible.value = false;
    video.pause();
    video.cancelVideoFrameCallback(videoFrameCallbackHandle);
}

function togglePause(){
    video.paused ? play() : pause();
}

defineExpose({play, stop, load, unload, getElementRef});


function toggleMute(){
    muted.value = !muted.value;
}

function load(){
    if(!video.src){
        video.src = video.dataset.src;
        seeker.max = props.video.duration;
        seeker.value = video.currentTime;
        video.currentTime = startOffset;
        video.requestVideoFrameCallback((now, metadata) => {
            updateCanvas(now, metadata, true);
        });
        console.log("loaded " + getShortTitle());
    }
}

function unload(){
    if(video.src){
        video.removeAttribute("src");
        console.log("unloaded " + getShortTitle());
    }
}

function updateCanvas(now, metadata, runOnce){
    if(!videoRef.value) return;

    const ctx = canvas.getContext("2d");

    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;

    const w = window.innerHeight * (metadata.width / metadata.height);
    const h = window.innerHeight;
    const x = 0 - ((w - window.innerWidth) / 2);
    const y = 0;
    ctx.filter = "blur(64px)";
    ctx.drawImage(videoRef.value, x, y, w, h);
    ctx.filter = "none";

    const sw = w * scale;
    const sh = h * scale;
    const sx = 0 - ((sw - window.innerWidth) / 2);
    const sy = 0 - ((sh - window.innerHeight) / 2);
    ctx.drawImage(videoRef.value, sx, sy, sw, sh);

    if(runOnce) return;

    video.requestVideoFrameCallback(updateCanvas)
}

</script>