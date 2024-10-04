<template>
    <div ref="scrollContainerRef" class="h-dscreen bg-white overflow-x-hidden overflow-y-scroll snap-y snap-mandatory ">
        <VideoContainer ref="videoContainersRef" v-for="video in allVideos" :key="video.url" :video="video"></VideoContainer>
    </div>
    <div class="absolute top-0 left-0 z-50 text-white cursor-pointer p-6 text-2xl" @click="openMenu()">☰</div>
    <div :class="{'left-0': channels.visible, 'left-[-100%]': !channels.visible}" class="absolute top-0 z-50 text-white flex flex-col h-dscreen transition-all">
        <div class="text-lg font-bold flex justify-between bg-black">
            <span class="p-4">JW TIKTOK</span>
            <span class="p-4" @click="closeMenu()">✖</span>
        </div>
        <hr>
        <div class="flex-1 cursor-pointer text-lg p-4 border-b border-white bg-black bg-opacity-80 flex items-center" v-for="channel in channels.list" @click.stop="channels.load(channel)">
            {{ channel.text }}
        </div>
    </div>
</template>

<script setup>

import VideoContainer from './VideoContainer.vue';
import { ref, reactive, onMounted, nextTick } from 'vue';

function openMenu(){
    channels.visible = true;
}

function closeMenu(){
    channels.visible = false;
}

const channels = reactive({
    visible: false,
    list: [
        { text: "Morning Worship", url: "https://b.jw-cdn.org/apis/mediator/v1/categories/E/VODPgmEvtMorningWorship?detailed=1&mediaLimit=0&clientType=www"},
        { text: "Gilead Graduation", url: "https://b.jw-cdn.org/apis/mediator/v1/categories/E/VODPgmEvtGilead?detailed=1&mediaLimit=0&clientType=www"},
        { text: "Bible Principles", url: "https://b.jw-cdn.org/apis/mediator/v1/categories/E/VODBiblePrinciples?detailed=1&mediaLimit=0&clientType=www"},
        { text: "Truth Transforms Lives", url: "https://b.jw-cdn.org/apis/mediator/v1/categories/E/VODIntExpTransformations?detailed=1&mediaLimit=0&clientType=www"},
        { text: "JW Broadcasting Talks", url: "https://b.jw-cdn.org/apis/mediator/v1/categories/E/StudioTalks?detailed=1&mediaLimit=0&clientType=www"},
        { text: "Convention Music Presentations", url: "https://b.jw-cdn.org/apis/mediator/v1/categories/E/VODConvMusic?detailed=1&mediaLimit=0&clientType=www"},
        { text: "Annual Meetings", url: "https://b.jw-cdn.org/apis/mediator/v1/categories/E/VODPgmEvtAnnMtg?detailed=1&clientType=www"}
    ],
    active: null,
    async load(channel){
        //these lines are specifically for when we change the channel
        if(videoContainers) videoContainers.forEach((v) => v.stop())
        closeMenu();
        //

        this.active = channel;
        
        await loadVideos(); // need to load the videos so videoContainersRef actually has something
        console.log(allVideos.value);
        await nextTick(); // allow for DOM to update
        console.log(videoContainers);
        
        scrollContainer = scrollContainerRef.value;
        videoContainers = videoContainersRef.value; //  now get the videoContainers
        console.log(videoContainers);
        
        scrollContainer.scrollTop = 0;
        currentVideoIndex = 0;

        videoContainers[0].load();
        videoContainers[0].play();

        videoContainers[1].load();
    }
})

const allVideos = ref([]);

const scrollContainerRef = ref();
let scrollContainer;

const videoContainersRef = ref();
let videoContainers;

let currentVideoIndex = 0;
function play(videoIndex){
    if(videoIndex === currentVideoIndex) return;

    currentVideoIndex = videoIndex;

    videoContainers.forEach((video, i) => {
        if(videoIndex === i){
            video.play(true)
        }else{
            video.stop();

            if(videoIndex === i + 1 || videoIndex === i - 1){
                video.load();
            }else{
                video.unload();
            }
        }
    })
}

function getVisibleVideoIndex(){
    // have to get this dynamically because height changes depending on presence of address bar on mobile
    const videoHeight = videoContainers[0].getElementRef.value.getBoundingClientRect().height;
    return Math.round(scrollContainer.scrollTop / videoHeight);
}

async function loadVideos(){
    allVideos.value = [];
    await nextTick();
    const response = await fetch(channels.active.url);
    const json = await response.json();
    console.log(json);

    const all = json.category.media.map(v => {
        return {
            src: v.files.find(r => r.label === '360p').progressiveDownloadURL,
            title: v.title,
            url: "https://www.jw.org/finder?srcid=share&wtlocale=E&lank=" + v.languageAgnosticNaturalKey,
            duration: v.duration,
        }
    });

    shuffleArray(all);

    allVideos.value = all;
}

function shuffleArray(array) {
    for (let i = array.length - 1; i >= 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
    }
}

onMounted(async () => {
    await channels.load(channels.list[0]);

    let scrollTimeout;

    let touching = false;
    let scrolling = false;
    scrollContainer.addEventListener("touchstart", (e) => {
        touching = true;
    })
    scrollContainer.addEventListener("touchend", (e) => {
        touching = false;
        setTimeout(() => {
            // sometimes after a swipe, if you touch, the container is still snapping
            // when you let go, it's not scrolling anymore, it should play the current video
            if(!scrolling){
                play(getVisibleVideoIndex());
            }
            // if it is still scrolling, let the scrolling handler play the video
        }, 100);
    })

    scrollContainer.addEventListener('scroll', (e) => {
        scrolling = true;
        clearTimeout(scrollTimeout);

        scrollTimeout = setTimeout(() => {
            // scrolling stopped - either by touch or by landing
            scrolling = false;
            if(touching){
                // by touch. abandon and reset
                return false;
            }else{
                // by landing. play the video
                play(getVisibleVideoIndex());
            }
        }, 100);
    })
})

</script>

<style>

html {
    user-select: none;
}

</style>