<template>
    <div
        @mousewheel="onMouseWheel"
        style="
            width: 800px;
            height: 800px;
            border: 1px solid red;
            position: relative;
            overflow: hidden;
        "
    >
        <svg id="svg1" width="8800" height="8800"></svg>
    </div>
</template>

<script setup>
import { onMounted } from 'vue'
import { DagreLayout } from '@antv/layout'

let s = null
let g = null
let nodes = {}

Snap.plugin(function (Snap, Element, Paper, global, Fragment) {
    function dragStart(x, y, e) {
        this.current_transform = this.transform()
    }

    function dragMove(dx, dy, x, y, e) {
        this.transform(this.current_transform + 'T' + dx + ',' + dy)

        this.updatePaths()
    }

    function dragEnd(e) {
        this.current_transform = this.transform()
    }

    function updatePaths() {
        this.rect.forEach((rect) => {
            for (let key in rect.paths) {
                rect.paths[key][0].attr({
                    path: rect.getPathString(
                        rect.paths[key][1],
                        rect.paths[key][2]
                    ),
                })
            }
        })
    }

    function getCoordinates() {
        let group = this.group
        let index = this.index

        return [group.matrix.e, group.matrix.f + 50 + 30 * index + 15]
    }

    function drawLineArrow(x1, y1, x2, y2) {
        var path
        var slopy, cosy, siny
        var Par = 10.0
        var x3, y3
        slopy = Math.atan2(0, x1 > x2 - 200 ? 10 : -10)
        cosy = Math.cos(slopy)
        siny = Math.sin(slopy)
        //path = 'M' + x1 + ',' + y1 + ' L' + x2 + ',' + y2

        path = 'M' + x1 + ',' + y1 + ' L' + x2 + ',' + y2
        x3 = x2 // Number(x1) + Number(x2)
        y3 = y2 // Number(y1) + Number(y2)
        path += ' M' + x3 + ',' + y3
        path +=
            ' L' +
            (Number(x3) + Number(Par * cosy - (Par / 2.0) * siny)) +
            ',' +
            (Number(y3) + Number(Par * siny + (Par / 2.0) * cosy))
        path +=
            ' M' +
            (Number(x3) +
                Number(Par * cosy + (Par / 2.0) * siny) +
                ',' +
                (Number(y3) - Number((Par / 2.0) * cosy - Par * siny)))
        path += ' L' + x3 + ',' + y3
        return path
    }

    function getPathString(obj, direction) {
        var p1 = this.getCoordinates()
        var p2 = obj.getCoordinates()

        if (direction === 'ts') {
            ;[p1, p2] = [p2, p1]
        }

        let path
        let slopy, cosy, siny
        let Par = 10.0
        let [x1, y1] = p1
        let [x2, y2] = p2
        let x3, y3
        slopy = Math.atan2(0, p1[0] > p2[0] ? 10 : -10)
        cosy = Math.cos(slopy)
        siny = Math.sin(slopy)

        if (p1[0] > p2[0]) {
            x2 += 200

            path = 'M' + x1 + ',' + y1
            path += ' L' + (x1 - 10) + ',' + y1
            path += ' L' + (x2 + 10) + ',' + y2
            path += ' L' + x2 + ',' + y2
        } else {
            x1 += 200

            path = 'M' + x1 + ',' + y1
            path += ' L' + (x1 + 10) + ',' + y1
            path += ' L' + (x2 - 10) + ',' + y2
            path += ' L' + x2 + ',' + y2
        }

        x3 = x2 // Number(x1) + Number(x2)
        y3 = y2 // Number(y1) + Number(y2)
        path += ' M' + x3 + ',' + y3
        path +=
            ' L' +
            (Number(x3) + Number(Par * cosy - (Par / 2.0) * siny)) +
            ',' +
            (Number(y3) + Number(Par * siny + (Par / 2.0) * cosy))
        path +=
            ' M' +
            (Number(x3) +
                Number(Par * cosy + (Par / 2.0) * siny) +
                ',' +
                (Number(y3) - Number((Par / 2.0) * cosy - Par * siny)))
        path += ' L' + x3 + ',' + y3

        return path
    }

    function addPath(obj) {
        var id = obj.id
        var path = this.paper
            .path(this.getPathString(obj, 'st'))
            .attr({ fill: 'none', stroke: 'green', strokeWidth: 1 })

        path.hover(
            function () {
                this.animate(
                    {
                        strokeWidth: 2,
                    },
                    300
                )
            },
            function () {
                this.animate(
                    {
                        strokeWidth: 1,
                    },
                    300
                )
            }
        )

        let title = Snap.parse(
            `<title>${this.nodeData.name + ' -> ' + obj.nodeData.name}</title>`
        )

        path.append(title)

        path.dblclick(() => {
            console.log(this.nodeData, obj.nodeData)
        })

        g.add(path)

        this.paths[id] = [path, obj, 'st']
        obj.paths[this.id] = [path, this, 'ts']
    }

    function removePath(obj) {
        var id = obj.id
        if (this.paths[id] != null) {
            this.paths[id][0].remove()
            this.paths[id][1] = null
            delete this.paths[id]

            obj.paths[this.id][1] = null
            delete obj.paths[this.id]
        }
    }

    Paper.prototype.childRect = function (x, y, node) {
        let rg = s.g()

        rg.rect = []

        let titleRect = this.rect(0, 0, 200, 50, 0).attr({
            fill: '#00ffff',
            stroke: '#ccc',
            strokeWidth: 1,
        })

        let titleText = this.text(100, 32, node.name).attr({
            width: 196,
            cursor: 'default',
            'text-anchor': 'middle',
        })

        titleRect.mousedown((evt) => {
            evt.stopPropagation()
        })

        titleText.mousedown((evt) => {
            evt.stopPropagation()
        })

        titleRect.drag(dragMove.bind(rg), dragStart.bind(rg), dragEnd.bind(rg))
        titleText.drag(dragMove.bind(rg), dragStart.bind(rg), dragEnd.bind(rg))

        rg.add(titleRect)
        rg.add(titleText)

        node.children.forEach((child, index) => {
            let childRect = this.rect(0, 50 + index * 30, 200, 30, 0).attr({
                fill: '#eee',
                stroke: '#ccc',
                strokeWidth: 1,
            })

            let childText = this.text(
                100,
                50 + 22 + index * 30,
                child.name
            ).attr({
                width: 196,
                cursor: 'default',
                'text-anchor': 'middle',
            })

            childRect.paths = {}
            /* childRect.updatePaths = updatePaths */
            childRect.getCoordinates = getCoordinates
            childRect.getPathString = getPathString
            childRect.addPath = addPath
            childRect.removePath = removePath
            childRect.nodeData = node
            childRect.group = rg
            childRect.index = index

            nodes[child.id] = {
                rect: childRect,
                data: node,
            }

            rg.add(childRect)
            rg.add(childText)

            rg.rect.push(childRect)
        })

        rg.transform('T' + x + ',' + y)
        rg.updatePaths = updatePaths
        g.add(rg)
    }
})

let data = {
    nodes: [
        {
            id: '100',
            name: 'left_1',
            children: [
                { id: '100-1', name: 'left_1-1' },
                { id: '100-2', name: 'left_1-2' },
                { id: '100-3', name: 'left_1-3' },
                { id: '100-4', name: 'left_1-4' },
                { id: '100-5', name: 'left_1-5' },
            ],
        },
        {
            id: '101',
            name: 'left_2',
            children: [
                { id: '101-1', name: 'left_2-1' },
                { id: '101-2', name: 'left_2-2' },
                { id: '101-3', name: 'left_2-3' },
                { id: '101-4', name: 'left_2-4' },
                { id: '101-5', name: 'left_2-5' },
            ],
        },

        {
            id: '0',
            name: 'center_1',
            children: [
                { id: '0-1', name: 'center_1-1' },
                { id: '0-2', name: 'center_1-2' },
                { id: '0-3', name: 'center_1-3' },
                { id: '0-4', name: 'center_1-4' },
                { id: '0-5', name: 'center_1-5' },
            ],
        },

        {
            id: '1',
            name: 'right_1',
            children: [
                { id: '1-1', name: 'right_1-1' },
                { id: '1-2', name: 'right_1-2' },
                { id: '1-3', name: 'right_1-3' },
                { id: '1-4', name: 'right_1-4' },
                { id: '1-5', name: 'right_1-5' },
            ],
        },
        {
            id: '2',
            name: 'right_2',
            children: [
                { id: '2-1', name: 'right_2-1' },
                { id: '2-2', name: 'right_2-2' },
                { id: '2-3', name: 'right_2-3' },
                { id: '2-4', name: 'right_2-4' },
                { id: '2-5', name: 'right_2-5' },
            ],
        },
    ],
    edges: [
        {
            source: '100',
            target: '0',
            relations: [
                { from: '100-1', to: '0-1' },
                { from: '100-2', to: '0-2' },
            ],
        },

        {
            source: '101',
            target: '0',
            relations: [
                { from: '101-4', to: '0-4' },
                { from: '101-5', to: '0-5' },
            ],
        },

        {
            source: '0',
            target: '1',
            relations: [
                { from: '0-1', to: '1-1' },
                { from: '0-2', to: '1-2' },
            ],
        },
        {
            source: '0',
            target: '2',
            relations: [
                { from: '0-4', to: '2-4' },
                { from: '0-5', to: '2-5' },
            ],
        },
    ],
}

onMounted(() => {
    s = Snap('#svg1')
    g = s.g()

    let isdown = false
    let startX = 0
    let startY = 0
    let distanX = 0
    let distanY = 0
    let moveTransform = ''

    s.mousedown((evt) => {
        isdown = true

        startX = evt.x
        startY = evt.y

        moveTransform = g.transform()
    })

    s.mousemove((evt) => {
        if (!isdown) {
            return
        }

        let disX = evt.x - startX
        let disY = evt.y - startY

        g.transform(moveTransform + 'T' + disX + ',' + disY)
    })

    s.mouseup((evt) => {
        if (!isdown) {
            return
        }

        isdown = false

        distanX += evt.x - startX
        distanY += evt.y - startY

        moveTransform = g.transform()
    })

    const dagreLayout = new DagreLayout({
        type: 'dagre',
        rankdir: 'RL',
        ranksep: 100,
        nodesep: 50,
        controlPoints: true,
        nodesepFunc: (d) => {
            return d.children.length * 30
        },
    })

    let timer = setInterval(() => {
        if (dagreLayout.nodes && dagreLayout.nodes.length !== 0) {
            dagreLayout.nodes.forEach((node) => {
                s.childRect(node.x, node.y, node)
            })

            dagreLayout.edges.forEach((edge) => {
                edge.relations.forEach((relation) => {
                    let sNode = nodes[relation.from]
                    let tNode = nodes[relation.to]

                    sNode.rect.addPath(tNode.rect)
                })
            })

            clearInterval(timer)
        }
    }, 200)

    dagreLayout.layout(data)
})

let scaleSize = 0.1
const onMouseWheel = (evt) => {
    console.log(evt.deltaY)

    let scale = 1 + (evt.deltaY < 0 ? scaleSize : -scaleSize)

    let scaleTransform = g.transform()
    g.transform(scaleTransform + 'S' + scale)
}
</script>

<style>
#svg {
    width: 800px;
    height: 800px;
    border: 1px solid #ccc;
}
</style>
