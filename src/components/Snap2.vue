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
        <svg id="svg" width="8800" height="8800"></svg>
    </div>
</template>

<script setup>
import { onMounted } from 'vue'
import { DagreLayout } from '@antv/layout'

let s = null
let g = null

Snap.plugin(function (Snap, Element, Paper, global, Fragment) {
    function dragStart(x, y, e) {
        this.current_transform = this.transform()
        this.text.current_transform = this.text.transform()
    }

    function dragMove(dx, dy, x, y, e) {
        this.transform(this.current_transform + 'T' + dx + ',' + dy)
        this.text.transform(this.text.current_transform + 'T' + dx + ',' + dy)
        this.updatePaths()
    }

    function dragEnd(e) {
        this.current_transform = this.transform()
        this.text.current_transform = this.text.transform()
    }

    function updatePaths() {
        var key
        for (key in this.paths) {
            this.paths[key][0].attr({
                path: this.getPathString(
                    this.paths[key][1],
                    this.paths[key][2]
                ),
            })

            //this.paths[key][0].prependTo(this.paper)
        }
    }

    function getCoordinates() {
        return [
            this.matrix.e + this.node.width.baseVal.value / 2,
            this.matrix.f + this.node.height.baseVal.value / 2,
        ]
    }

    function getPathString(obj, direction) {
        var p1 = this.getCoordinates()
        var p2 = obj.getCoordinates()

        if (direction === 'ts') {
            ;[p1, p2] = [p2, p1]
        }

        if (p1[0] > p2[0] + 100) {
            p1[0] -= 50
            p2[0] += 50
        } else if (p1[0] + 100 < p2[0]) {
            p1[0] += 50
            p2[0] -= 50
        } else {
            if (p1[1] > p2[1]) {
                p1[1] -= 25
                p2[1] += 25
            } else {
                p1[1] += 25
                p2[1] -= 25
            }
        }

        return drawLineArrow(p1[0], p1[1], p2[0], p2[1])
    }

    function addPath(obj) {
        var id = obj.id
        var path = this.paper
            .path(this.getPathString(obj, 'st'))
            .attr({ fill: 'none', stroke: 'green', strokeWidth: 1 })
        // path.prependTo(this.paper)
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

    Paper.prototype.draggableRect = function (x, y, w, h) {
        var rect = this.rect(0, 0, w, h).transform('T' + x + ',' + y)
        rect.paths = {}
        rect.drag(dragMove, dragStart, dragEnd)
        rect.updatePaths = updatePaths
        rect.getCoordinates = getCoordinates
        rect.getPathString = getPathString
        rect.addPath = addPath
        rect.removePath = removePath

        return rect
    }
})

function drawLineArrow(x1, y1, x2, y2) {
    var path
    var slopy, cosy, siny
    var Par = 10.0
    var x3, y3
    slopy = Math.atan2(y1 - y2, x1 - x2)
    cosy = Math.cos(slopy)
    siny = Math.sin(slopy)
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

function wrap(text, maxWidth) {
    var textLength = text.node().getComputedTextLength(),
        text = text.text()
    while (textLength > maxWidth && text.length > 0) {
        text = text.slice(0, -1)
        self.text(text + '...')
        textLength = self.node().getComputedTextLength()
    }
}

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

let nodes = {}

onMounted(() => {
    s = Snap('#svg')

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
            g = s.g().drag()

            dagreLayout.nodes.forEach((node) => {
                let rect = s.draggableRect(node.x, node.y, 100, 50, 10).attr({
                    fill: '#00ffff',
                })

                let text = s.text(node.x + 50, node.y + 32, node.name).attr({
                    width: 96,
                    'text-anchor': 'middle',
                })

                rect.text = text

                g.add(rect)
                g.add(text)

                rect.mousedown((evt) => {
                    evt.stopPropagation()
                })

                text.mousedown((evt) => {
                    evt.stopPropagation()
                })

                nodes[node.id] = {
                    data: node,
                    rect: rect,
                }
            })

            dagreLayout.edges.forEach((edge) => {
                let sNode = nodes[edge.source]
                let tNode = nodes[edge.target]

                sNode.rect.addPath(tNode.rect)
            })

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
