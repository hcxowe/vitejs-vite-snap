<template>
    <button @click="type = 1">类型</button>
    <button @click="type = 2">模块</button>
    <Snap v-show="type == 1" />
    <Snap1 v-show="type == 2" />
</template>

<script setup>
import { ref } from 'vue'
import Snap from './components/Snap3.vue'
import Snap1 from './components/Snap2.vue'

const type = ref(1)
</script>
<style>
#app {
    flex: 1;
}
</style>
