
# 菜单

> 菜单指的是右键菜单

## 启用

引入组件，启用默认菜单

```ts
import LogicFlow from '@logicflow/core';
import { Menu } from '@logicflow/extension';
import '@logicflow/extension/lib/style/index.css'

LogicFlow.use(Menu);
```

`Menu`组件支持菜单包括节点右键菜单、边右键菜单、画布右键菜单，默认情况下，`Menu`在各个菜单内置了以下功能。

- 节点右键菜单(nodeMenu)： 删除、复制、编辑文案
- 边右键菜单(edgeMenu)：删除、编辑文案
- 画布右键菜单(graphMenu)：无

## 菜单配置项

菜单中的每一项功能，可以用一条配置进行表示。具体字段如下
|字段|类型| 作用| 是否必须|描述|
|:--|:--|:-|:--|:-|
|text|string|文案||菜单项展示的文案|
|className|string|class名称||每一项默认class为lf-menu-item，设置了此字段，calss会在原来的基础上添加className。|
|icon|boolean|是否创建icon的span展位||如果简单的文案不能丰富表示菜单，可以加个icon设置为true,对应的菜单项会增加class为lf-menu-icon的span，通过为其设置背景的方式，丰富菜单的表示，一般与className配合使用。|
|callback|Function|点击后执行的回调|✅|三种菜单回调中分别可以拿到节点数据/边数据/事件信息。|

节点右键菜单删除功能，示例如下：

```ts
{
  text: '删除',
  callback(node) {
    // node为该节点数据
    lf.deleteNode(node.id);
  },
},
```

## 追加菜单选项

通过`lf.extension.menu.addMenuConfig`方法可以在原有菜单的基础上追加新的选项，具体配置示例如下：

```ts
import LogicFlow from '@logicflow/core';
import { Menu } from '@logicflow/extension';

// 注册组件
LogicFlow.use(Menu);
// 实例化 Logic Flow
const lf = new LogicFlow({
  container: document.getElementById('app');
});
// 为菜单追加选项（必须在 lf.render() 之前设置）
lf.extension.menu.addMenuConfig({
  nodeMenu: [
    {
      text: '分享',
      callback() {
        alert('分享成功！');
      }
    },
    {
      text: '属性',
      callback(node: any) {
        alert(`
          节点id：${node.id}
          节点类型：${node.type}
          节点坐标：(x: ${node.x}, y: ${node.y})`
        );
      },
    },
  ],
  edgeMenu: [
    {
      text: '属性',
      callback(edge: any) {
        alert(`
          边id：${edge.id}
          边类型：${edge.type}
          边坐标：(x: ${edge.x}, y: ${edge.y})
          源节点id：${edge.sourceNodeId}
          目标节点id：${edge.targetNodeId}`
        );
      },
    },
  ],
  graphMenu: [
    {
      text: '分享',
      callback() {
        alert('分享成功！');
      }
    },
  ],
});
lf.render();
```

## 重置菜单

如果默认菜单中存在不需要的选项，或者无法满足需求，可以通过`lf.setMenuConfig`重置菜单，更换为自定义菜单。

```ts
lf.extension.menu.setMenuConfig({
    nodeMenu: [
      {
        text: '删除',
        callback(node) {
          lf.deleteNode(node.id);
        },
      },
    ], // 覆盖默认的节点右键菜单
    edgeMenu: false, // 删除默认的边右键菜单
    graphMenu: [],  // 覆盖默认的边右键菜单，与false表现一样
  });
```

## 指定类型元素配置菜单

除了上面的为所有的节点、元素、画布自定义通用菜单外，还可以使用`lf.setMenuByType`为指定类型的节点或边定义菜单。

```ts
lf.extension.menu.setMenuByType({
  type: 'bpmn:startEvent',
  menu: [
    {
      text: '分享111',
      callback() {
        console.log('分享成功222！');
      }
    },
  ]
})
```

## 指定业务状态设置菜单

除了上面的为某种类型元素设置菜单外，还可以在自定义元素的时候，为节点处于不同业务状态下设置菜单。

- 通过自定义节点，设置其menu，从而为节点设置定制的自定义菜单。
- 由于自定义的model中可能无法直接拿到lf实例对象，此时可以通过`this.graphModel`拿到graphModel对象。graphModel对象详细说明请参考API/graphModel。
- 如果还希望在点击菜单后进行业务处理，可以通过`graphModel`的`eventCenter`发送自定义事件，然后自己在`lf`实例上监听此事件。
- 优先级：指定业务状态设置菜单 > 指定类型元素配置菜单 > 通用菜单配置 > 默认菜单。

```ts
// customNode.ts
import { RectNode, RectNodeModel } from '@logicflow/core';

class CustomeModel extends RectNodeModel {
  setAttributes() {
    this.stroke = '#1E90FF';
    this.fill = '#F0F8FF';
    this.radius = 10;
    const { properties: { isDisabledNode } } = this;
    if (!isDisabledNode) {
      // 单独为非禁用的元素设置菜单。
      this.menu = [
        {
          className: 'lf-menu-delete',
          icon: true,
          callback: (node) => {
            this.graphModel.deleteNode(node.id);
            this.graphModel.eventCenter.emit('custom:event', node);
          },
        },
        {
          text: 'edit',
          className: 'lf-menu-item',
          callback: (node) => {
            this.graphModel.setElementStateById(node.id, 2);
          },
        },
        {
          text: 'copy',
          className: 'lf-menu-item',
          callback: (node) => {
            this.graphModel.cloneNode(node.id);
          },
        },
      ];
    }
  }
}
// index.js
import { RectNode, CustomeModel } from './custom.ts';

lf.register({
  type: 'custome_node',
  view: RectNode,
  model: CustomeModel,
})

lf.on('custom:event', (r) => {
  console.log(r)
});
  ```

- 通过自定义边，设置其menu，从而为边设置定制的自定义菜单

```ts
// custom.ts
import { PolylineEdge, PolylineEdgeModel } from '@logicflow/core';
class CustomModel extends PolylineEdgeModel {
  setAttributes() {
    // 右键菜单
    this.menu = [
      {
        className: 'lf-menu-delete',
        icon: true,
        callback(edge) {
          const comfirm = window.confirm('你确定要删除吗？');
          comfirm && this.graphModel.deleteEdge(edge.id);
        },
      },
    ];
  }
}
// index.ts
lf.register({
  type: 'custome_edge',
  view: PolylineEdge,
  model: CustomeModel,
});
// 设置默认边的类型为自定义边类型
lf.setDefaultEdgeType('custome_edge');
```

```css
// css 设置
.lf-menu-delete .lf-menu-item-icon{
  display: inline-block;
  width: 20px;
  height: 20px;
  background: url('./delete.png') no-repeat;
  background-size: 20px;
}
```

### 菜单样式

根据菜单结构中的class覆盖原有样式，设置符合宿主风格的样式。

- 菜单：lf-menu
- 菜单项：lf-menu-item、用户自定义的className
- 菜单项-文案：lf-menu-item-text
- 菜单项-文案：lf-menu-item-icon,需要将菜单项配置icon设置为true
通过对设置这些class，可以覆盖默认样式，美化字体颜色，设置菜单项icon等。

注意，以上介绍的菜单配置必须在 `lf.render()`之前调用。

### 示例

<iframe src="https://codesandbox.io/embed/dazzling-hypatia-en8s9?fontsize=14&hidenavigation=1&theme=dark&view=preview"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="logicflow-base16"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>