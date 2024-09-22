<template>
    <div class="organization">
        <div class="container" id="container"></div>
    </div>
</template>
<script lang="ts" setup>
    import {
        defineComponent,
        onMounted
    } from 'vue'
    import * as G6 from '@antv/g6';

    const init = () => {

        // const data = {
        //     nodes: [{
        //             id: 'node1',
        //             label: '主节点',
        //         },
        //         {
        //             id: 'node2',
        //             label: '子节点1',
        //             parentId: 'node1'
        //         },
        //         {
        //             id: 'node3',
        //             label: '子节点2',
        //             parentId: 'node1'
        //         },
        //         // ... 可以添加更多子节点
        //     ],
        //     edges: [{
        //             source: 'node1',
        //             target: 'node2'
        //         },
        //         {
        //             source: 'node1',
        //             target: 'node3'
        //         },
        //         // ... 可以添加更多子节点的连线
        //     ]
        // };

        const data = {
            // 定义图数据
            nodes: [{
                    id: 'node1',
                    label: '节点1',
                },
                {
                    id: 'node2',
                    label: '节点2',
                },
                // ... 更多节点数据
            ],
            edges: [
                // 定义边数据
            ]
        };

        const graph = new G6.Graph({
            container: 'mountNode', // 指定图的容器
            width: 800, // 图的宽度
            height: 600, // 图的高度
            defaultNode: {
                type: 'custom-node', // 自定义节点类型
            },
            // 自定义节点的布局
            layout: (data) => {
                const nodes = data.nodes;
                let y = 0;
                nodes.forEach((node, i) => {
                    node.x = 50; // 节点的x坐标，可以根据需要调整
                    node.y = y; // 节点的y坐标，按顺序递增
                    y += 50; // 节点之间的垂直距离，可以根据需要调整
                });
            },
        });

        graph.data(data);
        graph.render();

    }
    onMounted(() => {

        setTimeout(() => {
            init()
        }, 1000);
    })
</script>

<style lang="scss" scoped>
    .container {
        width: 100%;
        height: 100vh;
    }
</style>