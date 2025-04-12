<template>
    <div class="toc-tree">
        <h2 class="toc-tree__title">{{ lectitle }}</h2>

        <div class="toc-tree__section" v-for="(section, index) in toc" :key="index">
            <span :class="'section-title ' + (path.indexOf(section) != -1 ? 'here' : '')"
                @click="toggleSection(index)">{{
                    section.title }}</span>

            <div v-show="section.open" class="toc-tree__section-content">
                <div class="toc-tree__sequence" v-for="(sequence, seqIndex) in section.sequences" :key="seqIndex">
                    <span :class="'seq-title ' + (path.indexOf(sequence) != -1 ? 'here' : '')"
                        @click="toggleSequence(index, seqIndex)">{{ sequence.title }}</span>

                    <ul v-show="sequence.open">
                        <li class="toc-tree__sequence-item" v-for="(item, itemIndex) in sequence.items"
                            :key="itemIndex">
                            <BsFillCameraReelsFill class="icon" v-if="item.type == 'video'" />
                            <BsFileTextFill class="icon" v-if="item.type == 'problem'" />
                            <a :class="(path.indexOf(item) != -1 ? 'here' : '')"
                                @click="onchangepath([section, sequence, item])">{{ item.title }}</a>
                        </li>
                    </ul>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { defineProps, defineEmits } from 'vue';
import { APIROOT } from '../config.js';
import { BsFillCameraReelsFill, BsFileTextFill } from 'vue-icons-plus/bs';

const props = defineProps({
    lecid: String,
    modelValue: Object
});
const emit = defineEmits(['update:modelValue']);

const lectitle = ref('Lecture Title');
const path = ref([]);
const toc = ref([]);

function toggleSection(index) {
    toc.value[index].open = !toc.value[index].open;
}

function toggleSequence(sectionIndex, seqIndex) {
    toc.value[sectionIndex].sequences[seqIndex].open = !toc.value[sectionIndex].sequences[seqIndex].open;
}

function onchangepath(newpath) {
    // Handle path change logic here
    console.log('Path changed to:', newpath);
    path.value = newpath;
    emit('update:modelValue', newpath[2]);
}

onMounted(() => {
    fetch(APIROOT + `/${props.lecid}/get_course_info`).then(resp => resp.json()).then(data => {
        toc.value = data["table_of_contents"].map(section => ({
            ...section,
            open: false,
            sequences: section.sequences.map(sequence => ({
                ...sequence,
                open: false
            }))
        }));
    }).catch(err => {
        console.error('Error fetching TOC:', err);
    });
});
</script>

<style scoped>
.toc-tree {
    text-align: left;
    padding: 10px;
    user-select: none;
}

.toc-tree__title {
    font-size: 1.5em;
    margin-bottom: 10px;
}

.toc-tree__section {
    /* margin-bottom: 10px; */
}

.section-title {
    display: block;
    box-sizing: border-box;
    font-size: 16px;
    line-height: 22px;
    font-family: 'Segoe UI', 'Segoe UI Web', 'wf_segoe-ui_normal', 'Helvetica Neue', 'BBAlpha Sans', 'S60 Sans', Arial, sans-serif;
    cursor: pointer;
    font-weight: normal;
    padding-top: 0;
    padding-bottom: 0;
    margin: 0;
    background-color: #fafafa;
    border-top: 1px solid #cecece;
    color: #1e1e1e;
    padding: 13px 18px 8px;
}

.section-title:hover {
    background-color: #fff;
}

.toc-tree__sequence {
    margin-left: 15px;
    margin-top: 5px;
    margin-bottom: 5px;
}

.toc-tree__section-content {
    margin-bottom: 20px;
}

.seq-title {
    margin-left: 4ch;
    cursor: pointer;
    font-family: 'Segoe UI', 'Segoe UI Web', 'wf_segoe-ui_normal', 'Helvetica Neue', 'BBAlpha Sans', 'S60 Sans', Arial, sans-serif;
    font-style: italic;
    text-decoration: underline;
    display: block;
}

.toc-tree__sequence-item {
    margin-left: 15px;
    list-style-type: none;
}

a {
    color: #006cac;
    text-decoration: none;
    cursor: pointer;
}

a:hover {
    /* color: #0056a1; */
    text-decoration: underline;
}

.here {
    font-weight: bold;
}

.icon {
    display: inline-block;
    height: 1rem;
}
</style>