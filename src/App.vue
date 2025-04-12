<template>
  <div class="tbar">
    <BiMenu @click="lbarOn = !lbarOn"/>
    <h1>{{ "mit700x" }}</h1>
  </div>

  <div class="lbar" v-show="lbarOn">
    <TocTree lecid="mit700x" v-model="page" />
  </div>
  <div class="mainbar">
    <VideoDisplay v-if="page && page.type == 'video'" lecid="mit700x" :vidInfo="page.content" />
  </div>
</template>

<script setup>
import { ref, watch } from 'vue';
const page = ref(null);
const lbarOn = ref(false);

watch(page, (newPage, oldPage) => {
  if (newPage) {
    lbarOn.value = false;
  }
}, { deep: true });
</script>

<script>
import TocTree from './components/TocTree.vue';
import VideoDisplay from './components/VideoDisplay.vue';
import { BiMenu } from 'vue-icons-plus/bi';

export default {
  name: 'App',
  components: {
    TocTree
  }
}
</script>

<style>
#app {
  margin-top: 3rem;
}

body {
  margin: 0;
}

.tbar {
  position: absolute;
  top: 0;
  left: 0;
  width: 100vw;
  padding: 5px;
  display: flex;
}

.tbar>h1 {
  margin: 0;
  position: absolute;
  top: 0;
  left: 50vw;
  transform: translateX(-50%);
  user-select: none;
}

.lbar {
  position: fixed;
  height: 100vh;
  overflow: auto;
  top: 3rem;
  left: 0;
  width: max-content;
  z-index: 500;
  background-color: white;
}

.mainbar {
  width: 100vw;
}
</style>
