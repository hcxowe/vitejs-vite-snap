<template>
    <div
        style="
            height: 800px;
            position: relative;
            background: #eee;
            overflow: auto;
        "
    >
        <div
            class="layout-item"
            v-for="(item, index) in nodes"
            :key="index"
            :style="{ left: `${item.x}px`, top: `${item.y}px` }"
        >
            {{ item.name }}
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { DagreLayout } from '@antv/layout'

const nodes = ref([])

let data = {
    nodes: [
        {
            id: '100',
            name: 'left_1',
            height: '200',
        },
        {
            id: '101',
            name: 'left_2',
        },
        {
            id: '102',
            name: 'left_3',
        },
        {
            id: '103',
            name: 'left_4',
        },

        {
            id: '0',
            name: 'center_1',
        },

        {
            id: '1',
            name: 'right_1',
            height: '200',
        },
        {
            id: '2',
            name: 'right_2',
        },
        {
            id: '3',
            name: 'right_3',
        },

        {
            id: '4',
            name: 'right_4',
        },
    ],
    edges: [
        {
            source: '100',
            target: '0',
        },
        {
            source: '101',
            target: '0',
        },
        {
            source: '102',
            target: '0',
        },
        {
            source: '103',
            target: '0',
        },
        {
            source: '103',
            target: '102',
        },
        {
            source: '0',
            target: '1',
        },
        {
            source: '0',
            target: '2',
        },
        {
            source: '0',
            target: '3',
        },
        {
            source: '0',
            target: '4',
        },
        {
            source: '4',
            target: '3',
        },

        {
            source: '3',
            target: '100',
        },

        {
            source: '2',
            target: '4',
        },
    ],
}

const dagreLayout = new DagreLayout({
    type: 'dagre',
    rankdir: 'RL',
    ranksep: 100,
    nodesep: 20,
    controlPoints: true,
    nodesepFunc: (d) => {
        return d.height || 10
    },
})

let timer = setInterval(() => {
    if (dagreLayout.nodes && dagreLayout.nodes.length !== 0) {
        nodes.value = dagreLayout.nodes

        console.log(dagreLayout)

        clearInterval(timer)
    }
}, 200)

dagreLayout.layout(data)
</script>

<style scoped>
.layout-item {
    position: absolute;
    width: 100px;
    padding: 10px;
    background: green;
}
</style>
