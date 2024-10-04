<template>
    <div ref="app" class="relative h-full">
        <div ref="scrollContainer" class="overflow-y-scroll snap-y snap-mandatory overflow-x-hidden h-full w-screen"
            style="scrollbar-width: none">
            <MyVideo ref="videoContainers" v-for="video in videoList" :video="video"
                class="outline snap-always snap-start" />
        </div>
        <div class="absolute z-50 bottom-0 right-0 mb-12 mr-8 text-white text-4xl" @click="toggleFullscreen()">
            ⛶
        </div>
    </div>
    <MyModal v-if="modal.visible" :message="modal.message" @modal-ok="modal.close()" />
</template>

<script setup>

import { onMounted, reactive, ref } from "vue";
import MyVideo from "./components/MyVideo.vue";
import MyModal from "./components/MyModal.vue";

const scrollContainer = ref();
const videoContainers = ref();
const app = ref();

const modal = reactive({
    visible: true,
    message: "Play the first video?",
    cb: null,
    open(cb) {
        this.cb = cb;
        this.visible = true;
    },
    close() {
        this.visible = false;
        this.cb();
    }
})

let videoIndex = 0;
let level = 0; // 0, 1, 2
let timeoutHandle = null;


const allVideos = [
    { src: "https://d34ji3l0qn3w2t.cloudfront.net/2fa175b9-33f5-4587-a072-085b8f115c7b/1/jwbvod24_E_10_r720P.mp4", url: "https://www.jw.org/finder?srcid=share&wtlocale=E&lank=pub-jwbvod24_10_VIDEO", title: "Gage Fleegle: Benefiting From Prohibitions (Eccl. 7:9)" },
    { src: "https://d34ji3l0qn3w2t.cloudfront.net/0142370f-ae79-4166-a746-481a5fce261b/1/jwbvod24_E_09_r360P.mp4", url: "https://www.jw.org/finder?srcid=share&wtlocale=E&lank=pub-jwbvod24_9_VIDEO", title: "Jonathan Smith: See the Bigger Picture (1 Sam. 17:42)" },
    { src: "https://d34ji3l0qn3w2t.cloudfront.net/3b5d7ae2-82a4-402a-a349-01cc0dc77ea3/2/jwb-099_E_07_r360P.mp4", url: "https://www.jw.org/finder?srcid=share&wtlocale=E&lank=pub-jwb-099_7_VIDEO", title: "Seth Hyatt: Speaking Truth Though Labeled as Deceivers (2 Cor. 6:8)" },
    { src: "https://d34ji3l0qn3w2t.cloudfront.net/db6ae25f-8784-4922-bfde-f0919410bbe4/1/jwbvod24_E_08_r360P.mp4", url: "https://www.jw.org/finder?srcid=share&wtlocale=E&lank=pub-jwbvod24_8_VIDEO", title: "Kenneth Godburn: Listen With Mildness (Jas. 1:19-21)" },
    { src: "https://d34ji3l0qn3w2t.cloudfront.net/2611d6fe-4202-456d-b6f3-62133e7d8955/1/jwbvod24_E_07_r360P.mp4", url: "https://www.jw.org/finder?srcid=share&wtlocale=E&lank=pub-jwbvod24_7_VIDEO", title: "Donald Gordon: “Pay Constant Attention to . . . Your Teaching” (1 Tim. 4:16)" },
    { src: "https://d34ji3l0qn3w2t.cloudfront.net/fb237fbc-4a17-4afb-bfcb-ec70a7282f38/1/jwbvod24_E_06_r360P.mp4", url: "https://www.jw.org/finder?srcid=share&wtlocale=E&lank=pub-jwbvod24_6_VIDEO", title: "Nicholas Ahladis: Jehovah Cares for the “Little Ones” (Matt. 18:10)" },
];

const videoList = ref(allVideos.slice(0, 2));

function toggleFullscreen() {
    document.fullscreenElement ? document.exitFullscreen() : app.value.requestFullscreen();
}

function updateMediaPool() {
    const previousVideoIndex = videoIndex;
    const previousLevel = level;

    // what level am I on?
    const y = scrollContainer.value.scrollTop; // 0 == 0; 768 == 1, 1536 == 2
    level = Math.round(y / window.innerHeight);

    // is this where I was?
    if (level === previousLevel) return;

    // stop the previous video
    videoContainers.value[previousLevel]?.stop();

    // reshuffle
    if (level > previousLevel) { // I went from level 1 to level 2 (or 0 to 1)
        videoIndex++;

        if(allVideos[videoIndex + 1]){
            videoList.value = [allVideos[videoIndex - 1], allVideos[videoIndex], allVideos[videoIndex + 1]]
            level = 1;
        }else{
            videoList.value = [allVideos[videoIndex - 2], allVideos[videoIndex - 1], allVideos[videoIndex]]
            level = 2;
        }
    } else
    if (level < previousLevel) { // I went from level 1 to level 0 (or 2 to 1)
        videoIndex--;

        if(allVideos[videoIndex - 1]){
            videoList.value = [allVideos[videoIndex - 1], allVideos[videoIndex], allVideos[videoIndex + 1]]
            level = 1;
        }else{
            videoList.value = [allVideos[videoIndex], allVideos[videoIndex + 1], allVideos[videoIndex + 2]]
            level = 0;
        }
    }

    scrollContainer.value.scrollTop = Math.round(level * window.innerHeight);

    // play the current video
    setTimeout(() => { videoContainers.value[level]?.play() }, 0);
}

onMounted(() => {

    modal.open(() => {
        //app.value.requestFullscreen();
        videoContainers.value[videoIndex]?.play();
    })

    scrollContainer.value.addEventListener("scroll", (e) => {
        clearTimeout(timeoutHandle);

        timeoutHandle = setTimeout(() => {
            updateMediaPool();
        }, 100)
    })
})

</script>

<style>

html, body {
    height: 100%;
}
</style>