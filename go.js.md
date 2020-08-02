# go.js

###### Diagrams
```
    var $ = go.GraphObject.make;
    var myDiagram =
        $(go.Diagram, "myDiagramDiv");
```

###### go.GraphObject.make 方法

其第一个参数必须是类类型，通常是GraphObject的子类。
GraphObject.make的 其他参数可能有几种类型：
1. 具有key/value对的普通JavaScript对象 设置属性值
2. 一个GraphObject，元素嵌套
3. 一个GoJS枚举值常数，它被用作对象的唯一属性的值被构造，可以接受这样的值
4. 一个字符串，代表某类属性，使用字符串值来设置Panel.type，Shape.figure和TextBlock.text属性
5. 一个RowColumnDefinition，用于描述在表的行或列面板小号
6. 一个JavaScript数组，其中包含GraphObject.make的参数，在从函数返回多个参数时很有用
7. 以适当方式用于构造对象的其他专用对象

###### 模型与Diagrams
MV结构 通过操作模型驱动视图修改

```
var $ = go.GraphObject.make;
// make 一个Diagram
var myDiagram =
  $(go.Diagram, "myDiagramDiv",
    { // enable Ctrl-Z to undo and Ctrl-Y to redo
      "undoManager.isEnabled": true
    });

// make 一个 Model
var myModel = $(go.Model);
// for each object in this Array, the Diagram creates a Node to represent it
myModel.nodeDataArray = [
  { key: "Alpha" },
  { key: "Beta" },
  { key: "Gamma" }
];
// 绑定model
myDiagram.model = myModel;
```
展示上面三个节点的同时，go也初始化了一些交互：

1. Click and drag the background in the above diagram to pan the view.
2. Click a node to select it, or press down on and drag a node to move it around.
3. To create a selection box, click and hold on the background, then start dragging.
4. Use CTRL-C and CTRL-V, or control-drag-and-drop, to make a copy of the selection.
5. Press the Delete key to delete selected nodes. (Read about more Keyboard Commands.)
6. On touch devices, press and hold to bring up a context menu. (Read about Context Menus.)
7.S ince the undo manager was enabled, CTRL-Z and CTRL-Y will undo and redo moves and copies and deletions.

###### 节点样式

1. Shape, to display pre-defined or custom geometry with colors
2. TextBlock, to display (potentially editable) text in various fonts
3. Picture, to display images, including SVG files
4. Panel, containers to hold a collection of other objects that can be positioned and sized in different manners according to the type of the Panel (like tables, vertical stacks, and stretching containers)

所有的基础节点类都继承自 GraphObject

模型-数据的绑定，需要先构建模板

```

myDiagram.nodeTemplate =
  $（go.Node，“ Vertical”，//节点（或任何面板）的第二个参数可以是面板类型
    / *在此处设置节点属性* / 
    { // Node.location点将位于每个中心节点
      locationSpot：go.Spot.Center
    }，

    / *在此处添加绑定* / 
    //示例节点绑定将Node.location设置为Node.data.loc的值
    new go.Binding（“ location”，“ loc”），

    / *添加包含在Node中的GraphObjects * / 
    //此Shape将垂直位于TextBlock的上方
    $ {go.Shape，
      “ RoundedRectangle”，//字符串参数可以命名一个预定义的图形 
      { / *在这里设置Shape属性* / }，
       //示例Shape绑定将Shape.figure设置为Node.data.fig的值
      new go.Binding（“ figure”，“ fig”）），

    $（go.TextBlock，
      “ default text”，   //字符串参数可以是初始文本字符串 
      { / *在这里设置TextBlock属性* / }，
       //示例TextBlock绑定将TextBlock.text设置为Node.data.text的值
      new go.Binding（“ text “））
  ）;

```

###### 模型的种类

>GraphLinksModel   关系关联

model.linkDataArray除了model.nodeDataArray。它包含一个JavaScript对象数组，每个对象通过指定“ to”和“ from”节点键来描述链接。

>TreeModel   父子关联

TreeModel的工作原理略有不同。代替维护链接数据的单独数组，而是通过为节点数据指定“父”来创建树模型中的链接。然后从该关联中创建链接。

###### Diagram中节点的布局
没有设置布局时，节点会重叠放置


###### Brush
用于给形状 边框增加渐变交过的样式刷，可以通过$实例化，可复用。

```
var violetbrush = $(go.Brush, "Linear", { 0.0: "Violet", 1.0: "Lavender" });
$(go.Node, "Auto",
      $(go.Shape, "RoundedRectangle",
        { fill: violetbrush }),
      $(go.TextBlock, "Hello!",
        { margin: 5 })
    )
```