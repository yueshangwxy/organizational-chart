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
        const data = {
            id: 'c0',
            level: 0,
            label: 'root',
            children: [{
                    id: 'c1',
                    level: 1,
                    label: '一级',
                    children: [{
                            id: 'c1-1',
                            level: 2,
                            label: '二级1',
                        },
                        {
                            id: 'c1-2',
                            level: 2,
                            label: '二级2',
                            children: [{
                                    id: 'c1-2-1',
                                    level: 3,
                                    label: '三级1',
                                    parentId: 'c1-2'
                                },
                                {
                                    id: 'c1-2-2',
                                    level: 3,
                                    label: '三级2',
                                    parentId: 'c1-2'
                                },
                            ],
                        },
                    ],
                },
                {
                    id: 'c2',
                    level: 1,
                    label: '一级2',
                    children: [{
                        id: 'c2-1',
                        level: 2,
                        label: '二级2',
                        children: [{
                                id: 'c2-1-1',
                                level: 3,
                                label: '三级1',
                                parentId: 'c2-1'
                            },
                            {
                                id: 'c2-1-2',
                                level: 3,
                                label: '三级2',
                                parentId: 'c2-1'
                            }
                        ]
                    }]
                },
                {
                    id: 'c22-1-2',
                    level: 3,
                    label: '三级11',
                    parentId: 'c0'
                },
                {
                    id: 'c23-1-2',
                    level: 3,
                    label: '三级22',
                    parentId: 'c0'
                }
            ],
        };

        G6.registerNode(
            'icon-node', {
                options: {
                    size: [60, 20],
                    stroke: '#91d5ff',
                    fill: '#91d5ff',
                },
                draw(cfg, group) {
                    const styles = this.getShapeStyle(cfg);
                    const {
                        labelCfg = {}
                    } = cfg;

                    const w = styles.width;
                    const h = styles.height;

                    const keyShape = group.addShape('rect', {
                        attrs: {
                            ...styles,
                            x: -w / 2,
                            y: -h / 2,
                        },
                    });

                    /**
                     * leftIcon 格式如下：
                     *  {
                     *    style: ShapeStyle;
                     *    img: ''
                     *  }
                     */
                    if (cfg.leftIcon) {
                        const {
                            style,
                            img
                        } = cfg.leftIcon;
                        group.addShape('rect', {
                            attrs: {
                                x: 1 - w / 2,
                                y: 1 - h / 2,
                                width: 38,
                                height: styles.height - 2,
                                fill: '#8c8c8c',
                                ...style,
                            },
                        });

                        group.addShape('image', {
                            attrs: {
                                x: 8 - w / 2,
                                y: 8 - h / 2,
                                width: 24,
                                height: 24,
                                img: img ||
                                    'https://g.alicdn.com/cm-design/arms-trace/1.0.155/styles/armsTrace/images/TAIR.png',
                            },
                            // must be assigned in G6 3.3 and later versions. it can be any string you want, but should be unique in a custom item type
                            name: 'image-shape',
                        });
                    }


                    if (cfg.label) {
                        group.addShape('text', {
                            attrs: {
                                ...labelCfg.style,
                                text: cfg.label,
                                x: 50 - w / 2,
                                y: 25 - h / 2,
                            },
                        });
                    }

                    return keyShape;
                },
                update: undefined,
            },
            'rect',
        );

        G6.registerEdge('flow-line', {
            draw(cfg, group) {
                const startPoint = cfg.startPoint;
                const endPoint = cfg.endPoint;


                const {
                    style
                } = cfg;
                const shape = group.addShape('path', {
                    attrs: {
                        stroke: style.stroke,
                        // endArrow: style.endArrow,
                        path: [
                            ['M', startPoint.x, startPoint.y],
                            ['L', startPoint.x, (startPoint.y + endPoint.y) / 2],
                            ['L', endPoint.x, (startPoint.y + endPoint.y) / 2],
                            ['L', endPoint.x, endPoint.y],
                        ],
                    },
                });

                return shape;
            },
        });

        const defaultStateStyles = {
            hover: {
                stroke: '#1890ff',
                lineWidth: 2,
            },
        };

        const defaultNodeStyle = {
            fill: '#91d5ff',
            stroke: '#40a9ff',
            radius: 5,
        };

        const defaultEdgeStyle = {
            stroke: '#91d5ff',
            endArrow: {
                path: 'M 0,0 L 12, 6 L 9,0 L 12, -6 Z',
                fill: '#91d5ff',
                d: -20,
            },
        };

        const defaultLayout = {
            type: 'compactBox',
            direction: 'TB',
            getId: function getId(d) {
                return d.id;
            },
            getHeight: function getHeight() {
                return 16;
            },
            getWidth: function getWidth() {
                return 16;
            },
            // 垂直
            getVGap: function getVGap(node) {
                let v = 20

                // if (node.level == 3) {
                //     // v = 0

                // }
                return v;
            },
            // 水平
            getHGap: function getHGap(node) {
                let v = 60
                // if (node.level == 3) {
                //     console.log(node);

                //     v = -18
                // }
                return v;
            },
        };

        const defaultLabelCfg = {
            style: {
                fill: '#000',
                fontSize: 12,
            },
        };

        const container = document.getElementById('container');
        const width = container.scrollWidth;
        const height = container.scrollHeight || 500;

        const minimap = new G6.Minimap({
            size: [150, 100],
        });
        const graph = new G6.TreeGraph({
            container: 'container',
            width,
            height,
            linkCenter: true,
            plugins: [minimap],
            modes: {
                default: ['drag-canvas', 'zoom-canvas'],
            },
            defaultNode: {
                type: 'icon-node',
                size: [120, 40],
                style: defaultNodeStyle,
                labelCfg: defaultLabelCfg,
            },
            defaultEdge: {
                type: 'flow-line',
                style: defaultEdgeStyle,
            },
            nodeStateStyles: defaultStateStyles,
            edgeStateStyles: defaultStateStyles,
            layout: defaultLayout,
            // layout: (data) => {
            //     console.log(data);

            //     const nodes = data.children;
            //     let y = 0;
            //     nodes.forEach((node, i) => {
            //         console.log(node, 111);

            //         node.x = 50; // 节点的x坐标，可以根据需要调整
            //         node.y = y; // 节点的y坐标，按顺序递增
            //         y += 50; // 节点之间的垂直距离，可以根据需要调整
            //     });
            // },

        });

        graph.data(data);
        graph.render();
        // graph.fitView();

        graph.layout(true)
        const nodes = graph.getNodes();
        nodes.forEach((node, index) => {
            const parentId: any = node.getModel().parentId;
            if (parentId) {
                const level: any = node.getModel().level;
                const parent = graph.findById(parentId);
                const parentModel = parent.getModel();
                const childModels = parentModel.children || [];
                // const x = parent.getBBox().minX + parent.getBBox().width / 2 - node.getBBox()
                //     .width / 2;
                const parentBox = parent.getBBox()
                const x = parentBox.minX + parentBox.width / 2;
                const y = parentBox.minY + index * 40 - 10; // 假设每个子节点的高度是40
                // childModels.forEach((item, idx) => {
                //     // let itemNode = graph.getNodes(item.id)
                //     const itemNode = graph.find('node', (n) => {
                //         return n.get('id') === item.id;
                //     });

                //     let yy = parentBox.y + (idx + 1) * 40 - 10
                //     console.log(yy, idx);

                //     itemNode.updatePosition({
                //         x,
                //         y
                //     })
                // })
                node.updatePosition({
                    x,
                    y
                });

            }

        });

        graph.on('afterlayout', () => {
            const nodes = graph.getNodes();
            console.log(nodes, 1111);

            // nodes.forEach((node, index) => {
            //     const parentId = node.getModel().parentId;
            //     if (parentId) {
            //         const parent = graph.findById(parentId);
            //         const parentModel = parent.getModel();
            //         const childModels = parentModel.children || [];
            //         const x = parent.getBBox().minX + parent.getBBox().width / 2 - node.getBBox()
            //             .width / 2;
            //         const y = parent.getBBox().minY + index * 50; // 假设每个子节点的高度是50
            //         node.updatePosition({
            //             x,
            //             y
            //         });
            //     }
            // });
        });


        graph.on('node:click', (evt) => {
            const {
                item,
                target
            } = evt;
            const targetType = target.get('type');
            const name = target.get('name');
            console.log(evt);


        });

        if (typeof window !== 'undefined')
            window.onresize = () => {
                if (!graph || graph.get('destroyed')) return;
                if (!container || !container.scrollWidth || !container.scrollHeight) return;
                graph.changeSize(container.scrollWidth, container.scrollHeight);
            };
    }
    onMounted(() => {

        setTimeout(() => {
            init()
        }, 1000);
    })
</script>

<style lang="scss" scoped>
    .container {
        width: 80%;
        height: 80vh;
    }
</style>