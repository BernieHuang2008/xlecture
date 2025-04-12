<template>
    <div class="video-container">
        <video ref="videoPlayer" class="video-player" controls
            :src="APIROOT + '/' + lecid + '/get_video/' + vidInfo.video" @play="onPlay" @pause="onPause"
            @ended="onEnded" @timeupdate="onTimeUpdate"></video>
        <div v-show="currentCC.length > 0" class="cc-container" ref="ccContainer" @mousedown="startDrag" :style="ccStyle">
            <div>
                <p :class="'cc cc-' + item.lang" v-for="(item, index) in currentCC" :key="index">
                    {{ item.text }}
                </p>
            </div>
        </div>
        <div id="fake-fullscreen-detector"></div>
    </div>
</template>

<script setup>
import { APIROOT } from '../config.js';
import { ref, reactive, onMounted, defineProps } from 'vue';

const props = defineProps({
    lecid: String,
    vidInfo: Object
});

const ccStyle = reactive({
    position: 'fixed',
    top: '100px',
    left: '100px',
    cursor: 'move'
});

let isDragging = false;
let dragOffset = { x: 0, y: 0 };

const videoPlayer = ref(null);
const ccData = reactive({
    en: [],
    zh: []
});
const currentCC = ref([]);

function startDrag(event) {
    isDragging = true;
    dragOffset.x = event.clientX - ccStyle.left.replace('px', '');
    dragOffset.y = event.clientY - ccStyle.top.replace('px', '');
    document.addEventListener('mousemove', onDrag);
    document.addEventListener('mouseup', stopDrag);
}

function onDrag(event) {
    if (isDragging) {
        ccStyle.left = `${event.clientX - dragOffset.x}px`;
        ccStyle.top = `${event.clientY - dragOffset.y}px`;
    }
}

function stopDrag() {
    isDragging = false;
    document.removeEventListener('mousemove', onDrag);
    document.removeEventListener('mouseup', stopDrag);
}

function onTimeUpdate() {
    const currentTime = videoPlayer.value.currentTime * 1000; // Convert to milliseconds
    var langs = ['en', 'zh'];
    var newCC = [];

    let lang = 'en';
    let cc = ccData[lang];

    for (var j = 0; j < cc.text.length; j++) {
        let start = cc.start[j];
        let end = cc.end[j];
        let text = cc.text[j];

        if (currentTime >= start && currentTime <= end) {
            for (let i = 0; i < langs.length; i++) {
                newCC[i] = {
                    text: ccData[langs[i]].text[j],
                    lang: langs[i]
                };
            }
            break;
        }
        if (start > currentTime) {
            break;
        }
    }

    currentCC.value = newCC
}

async function fetchCC() {
    try {
        const enResponse = await fetch(`${APIROOT}/${props.lecid}/get_cc/${props.vidInfo.cc.en}`);
        const zhResponse = await fetch(`${APIROOT}/${props.lecid}/get_cc/${props.vidInfo.cc.zh}`);
        ccData.en = await enResponse.json();
        ccData.zh = await zhResponse.json();
    } catch (error) {
        console.error('Error fetching CC data:', error);
    }
}

document.addEventListener('fullscreenchange', (event) => {
    const ffd = document.getElementById('fake-fullscreen-detector');

    if (document.fullscreenElement && event.target === ffd) {
        return false;
    }

    if (document.fullscreenElement) {
        // fullscreen mode turned on
        document.fullscreenElement.classList.toggle('fake-fullscreen');
        document.body.classList.toggle('fake-fullscreen');
        document.exitFullscreen();
        console.log(ffd, document.fullscreenElement);
        ffd.requestFullscreen();    // will occur permission error, ignore it.
    }
});

onMounted(() => {
    fetchCC();
});

function onPlay() {
    console.log('Video started playing');
}
function onPause() {
    console.log('Video paused');
}
function onEnded() {
    console.log('Video ended');
}
</script>

<script>
export default {
    name: 'VideoDisplay',
};
</script>

<style scoped>
.video-container {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100%;
    background-color: #000;
    position: relative;
}

.video-player {
    max-width: 100%;
    max-height: 100%;
}

.cc-container {
    background: rgba(0, 0, 0, 0.7);
    color: #fff;
    padding: 10px;
    border-radius: 5px;
    z-index: 1000;
    user-select: none;
    transform: translateX(-50%) translateY(-50%);
    width: max-content;
}

video.fake-fullscreen {
    position: fixed;
    top: 0;
    left: 0;
    height: 100vh;
    width: 100vw;
    z-index: 999 !important;
    background: black;
    pointer-events: auto;
    /* Ensures that mouse events are captured by this element */
}

video {
    z-index: 1;
}

.cc {
    margin: 2px 0;
}

.cc-zh {
    color: #858585;
}
</style>
<style>
body.fake-fullscreen {
    overflow: hidden;
    /* Prevents scrolling when in fake fullscreen */
}
</style>