<template>
    <div
        style="
            width: 800px;
            height: 800px;
            border: 1px solid red;
            position: relative;
            overflow: hidden;
        "
    >
        <svg id="svg" width="8800" height="8800"></svg>
    </div>
</template>

<script setup>
import { onMounted } from 'vue'
import { DagreLayout } from '@antv/layout'

let s = null

onMounted(() => {
    s = Snap('#svg')

    let data = {
        nodes: [
            {
                id: '100',
                name: 'left_1',
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
        nodesep: 50,
        controlPoints: true,
        nodesepFunc: (d) => {
            return 10
        },
    })

    let timer = setInterval(() => {
        if (dagreLayout.nodes && dagreLayout.nodes.length !== 0) {
            console.log(dagreLayout)

            let g = s.g()

            dagreLayout.nodes.forEach((node) => {
                let d = s.rect(node.x, node.y, 100, 50, 10).attr({
                    fill: '#00ffff',
                })

                let t = s.text(node.x + 25, node.y + 30, node.name)

                let g1 = s.g().drag()
                g1.add(d, t)

                g1.mousedown((evt) => {
                    evt.stopPropagation()
                })

                g.add(g1)
            })

            let isdown = false
            let startX = 0
            let startY = 0
            let distanX = 0
            let distanY = 0
            s.mousedown((evt) => {
                isdown = true

                startX = evt.x
                startY = evt.y
            })

            s.mousemove((evt) => {
                if (!isdown) {
                    return
                }

                let disX = evt.x - startX
                let disY = evt.y - startY

                var m = new Snap.Matrix()
                m.translate(disX + distanX, disY + distanY)
                g.transform(m)
            })

            s.mouseup((evt) => {
                if (!isdown) {
                    return
                }

                isdown = false

                distanX += evt.x - startX
                distanY += evt.y - startY
            })

            clearInterval(timer)
        }
    }, 200)

    dagreLayout.layout(data)
})
</script>

<style>
#svg {
    width: 800px;
    height: 800px;
    border: 1px solid #ccc;
}
</style>
