<a href="http://relation-graph.com" target="_blank"><img src="http://relation-graph.com/website/logo" width="60" /></a>

*relation-graph*
---

这是一个Vue关系图谱组件，可以展示如组织机构图谱、股权架构图谱、集团关系图谱等知识图谱，可提供多种图谱布局，包括树状布局、中心布局、力学布局自动布局等。

这个项目使用典型的vue编程方式，代码简单易懂。如果需要实现一些自定义的高级功能，你可以直接使用源码作为一个component放到你的项目中去用，轻松、任意的修改。

*详细使用方法、配置选项、在线demo，以及可视化的配置工具，可以访问这个网址：*

[http://relation-graph.com](http://relation-graph.com)

---
*快速使用：*
```shell script
npm install --save relation-graph
```
```vue
<template>
   <div>
     <div style="height:calc(100vh - 50px);">
        <RelationGraph ref="seeksRelationGraph" :options="graphOptions" :on-node-click="onNodeClick" :on-line-click="onLineClick" />
     </div>
   </div>
 </template>
 
 <script>
 import RelationGraph from 'relation-graph'
 export default {
   name: 'Demo',
   components: { RelationGraph },
   data() {
     return {
       graphOptions: {
         allowSwitchLineShape: true,
         allowSwitchJunctionPoint: true,
         defaultJunctionPoint: 'border'
         // 这里可以参考"Graph 图谱"中的参数进行设置:http://relation-graph.com/#/docs/graph
       }
     }
   },
   mounted() {
     this.showSeeksGraph()
   },
   methods: {
     showSeeksGraph() {
       var __graph_json_data = {
         rootId: 'a',
         nodes: [
            // node配置选项：http://relation-graph.com/#/docs/node
            // node支持通过插槽slot完全自定义，示例：http://relation-graph.com/#/demo/adv-slot
           { id: 'a', text: 'A', borderColor: 'yellow' },
           { id: 'b', text: 'B', color: '#43a2f1', fontColor: 'yellow' },
           { id: 'c', text: 'C', nodeShape: 1, width: 80, height: 60 },
           { id: 'e', text: 'E', nodeShape: 0, width: 150, height: 150 }
         ],
         links: [
            // link配置选项：http://relation-graph.com/#/docs/link
           { from: 'a', to: 'b', text: '关系1', color: '#43a2f1' },
           { from: 'a', to: 'c', text: '关系2' },
           { from: 'a', to: 'e', text: '关系3' },
           { from: 'b', to: 'e', color: '#67C23A' }
         ]
       }
       this.$refs.seeksRelationGraph.setJsonData(__graph_json_data, (seeksRGGraph) => {
         // Called when the relation-graph is completed 
       })
     },
     onNodeClick(nodeObject, $event) {
       console.log('onNodeClick:', nodeObject)
     },
     onLineClick(lineObject, $event) {
       console.log('onLineClick:', lineObject)
     }
   }
 }
 </script>
```
*上面代码的效果：*
![简单示例效果图](doc/relation-graph-simple.png)
![简单示例效果图](doc/images/d1.png)
![简单示例效果图](doc/images/d2.png)
![简单示例效果图](doc/images/d3.png)
![简单示例效果图](doc/images/d4.png)
![简单示例效果图](doc/images/d5.png)
![简单示例效果图](doc/images/d6.png)
![简单示例效果图](doc/images/d7.png)
![简单示例效果图](doc/images/d8.png)
![简单示例效果图](doc/images/d9.png)
![简单示例效果图](doc/images/d10.png)
![简单示例效果图](doc/images/d11.png)
![简单示例效果图](doc/images/d12.png)
![简单示例效果图](doc/images/d13.png)

*更多效果及使用方法：*
http://relation-graph.com
---

# 例子

图二换成折线图

```js
<template>
  <div>
    <div style="height:calc(100vh - 100px);">
      <SeeksRelationGraph ref="seeksRelationGraph" :options="graphOptions" />
      <!--    :on-node-click="onNodeClick"
        :on-line-click="onLineClick" -->
    </div>
  </div>
</template>

<script>
import SeeksRelationGraph from 'relation-graph';
const data = {
  rootId: 'a',
  nodes: [
    { id: 'a', text: 'a' },
    { id: 'b', text: 'b' },
    { id: 'b1', text: 'b1' },
    { id: 'b1-1', text: 'b1-1' },
    { id: 'b1-2', text: 'b1-2' },
    { id: 'b1-3', text: 'b1-3' },
    { id: 'b1-4', text: 'b1-4' },
    { id: 'b1-5', text: 'b1-5' },
    { id: 'b1-6', text: 'b1-6' },
    { id: 'b2', text: 'b2' },
    { id: 'b2-1', text: 'b2-1' },
    { id: 'b2-2', text: 'b2-2' },
    { id: 'b2-3', text: 'b2-3' },
    { id: 'b2-4', text: 'b2-4' },
    { id: 'b3', text: 'b3' },
    { id: 'b3-1', text: 'b3-1' },
    { id: 'b3-2', text: 'b3-2' },
    { id: 'b3-3', text: 'b3-3' },
    { id: 'b3-4', text: 'b3-4' },
    { id: 'b3-5', text: 'b3-5' },
    { id: 'b3-6', text: 'b3-6' },
    { id: 'b3-7', text: 'b3-7' },
    { id: 'b4', text: 'b4' },
    { id: 'b4-1', text: 'b4-1' },
    { id: 'b4-2', text: 'b4-2' },
    { id: 'b4-3', text: 'b4-3' },
    { id: 'b4-4', text: 'b4-4' },
    { id: 'b4-5', text: 'b4-5' },
    { id: 'b4-6', text: 'b4-6' },
    { id: 'b4-7', text: 'b4-7' },
    { id: 'b4-8', text: 'b4-8' },
    { id: 'b4-9', text: 'b4-9' },
    { id: 'b5', text: 'b5' },
    { id: 'b5-1', text: 'b5-1' },
    { id: 'b5-2', text: 'b5-2' },
    { id: 'b5-3', text: 'b5-3' },
    { id: 'b5-4', text: 'b5-4' },
    { id: 'b6', text: 'b6' },
    { id: 'b6-1', text: 'b6-1' },
    { id: 'b6-2', text: 'b6-2' },
    { id: 'b6-3', text: 'b6-3' },
    { id: 'b6-4', text: 'b6-4' },
    { id: 'b6-5', text: 'b6-5' }
  ],
  links: [
    { from: 'a', to: 'b', isHideArrow: true },
    { isHideArrow: true, from: 'b', to: 'b1' },
    { isHideArrow: true, from: 'b1', to: 'b1-1' },
    { isHideArrow: true, from: 'b1', to: 'b1-2' },
    { isHideArrow: true, from: 'b1', to: 'b1-3' },
    { isHideArrow: true, from: 'b1', to: 'b1-4' },
    { isHideArrow: true, from: 'b1', to: 'b1-5' },
    { isHideArrow: true, from: 'b1', to: 'b1-6' },
    { isHideArrow: true, from: 'b', to: 'b2' },
    { isHideArrow: true, from: 'b2', to: 'b2-1' },
    { isHideArrow: true, from: 'b2', to: 'b2-2' },
    { isHideArrow: true, from: 'b2', to: 'b2-3' },
    { isHideArrow: true, from: 'b2', to: 'b2-4' },
    { isHideArrow: true, from: 'b', to: 'b3' },
    { isHideArrow: true, from: 'b3', to: 'b3-1' },
    { isHideArrow: true, from: 'b3', to: 'b3-2' },
    { isHideArrow: true, from: 'b3', to: 'b3-3' },
    { isHideArrow: true, from: 'b3', to: 'b3-4' },
    { isHideArrow: true, from: 'b3', to: 'b3-5' },
    { isHideArrow: true, from: 'b3', to: 'b3-6' },
    { isHideArrow: true, from: 'b3', to: 'b3-7' },
    { isHideArrow: true, from: 'b', to: 'b4' },
    { isHideArrow: true, from: 'b4', to: 'b4-1' },
    { isHideArrow: true, from: 'b4', to: 'b4-2' },
    { isHideArrow: true, from: 'b4', to: 'b4-3' },
    { isHideArrow: true, from: 'b4', to: 'b4-4' },
    { isHideArrow: true, from: 'b4', to: 'b4-5' },
    { isHideArrow: true, from: 'b4', to: 'b4-6' },
    { isHideArrow: true, from: 'b4', to: 'b4-7' },
    { isHideArrow: true, from: 'b4', to: 'b4-8' },
    { isHideArrow: true, from: 'b4', to: 'b4-9' },
    { isHideArrow: true, from: 'b', to: 'b5' },
    { isHideArrow: true, from: 'b5', to: 'b5-1' },
    { isHideArrow: true, from: 'b5', to: 'b5-2' },
    { isHideArrow: true, from: 'b5', to: 'b5-3' },
    { isHideArrow: true, from: 'b5', to: 'b5-4' },
    { isHideArrow: true, from: 'b', to: 'b6' },
    { isHideArrow: true, from: 'b6', to: 'b6-1' },
    { isHideArrow: true, from: 'b6', to: 'b6-2' },
    { isHideArrow: true, from: 'b6', to: 'b6-3' },
    { isHideArrow: true, from: 'b6', to: 'b6-4' },
    { isHideArrow: true, from: 'b6', to: 'b6-5' }
  ]
};
export default {
  name: 'Demo',
  components: { SeeksRelationGraph },
  data() {
    return {
      currentCase: '横向树状图谱',
      isShowCodePanel: false,

      graphOptions: {
        backgrounImage: '',
        backgrounImageNoRepeat: false,
        layouts: [
          {
            label: '中心',
            layoutName: 'tree',
            layoutClassName: 'seeks-layout-center',
            defaultJunctionPoint: 'border',
            defaultNodeShape: 0,
            defaultLineShape: 1
          }
        ],
        defaultLineMarker: {
          markerWidth: 12,
          markerHeight: 12,
          refX: 6,
          refY: 6,
          data: 'M2,2 L10,6 L2,10 L6,6 L2,2'
        },
        allowShowMiniNameFilter: false,
        allowShowMiniToolBar: false,
        defaultNodeShape: 1,
        defaultExpandHolderPosition: 'hide',
        defaultLineShape: 4,
        allowShowMiniView: false,
        defaultJunctionPoint: 'tb',
        defaultShowLineLabel: false,
        hideNodeContentByZoom: false,
        disableDragNode: true,
        disableZoom: true
      }
    };
  },
  mounted() {
    this.showSeeksGraph();
  },
  methods: {
    showSeeksGraph() {
      var graphJsonData = data;
      this.$refs.seeksRelationGraph.setJsonData(
        graphJsonData
        // seeksRGGraph => {
        // 这些写上当图谱初始化完成后需要执行的代码
        // }
      );
    },
    onNodeClick(nodeObject, $event) {
      console.log('onNodeClick:', nodeObject, $event);
    },
    onLineClick(lineObject, $event) {
      console.log('onLineClick:', lineObject, $event);
    }
  }
};
</script>

```

