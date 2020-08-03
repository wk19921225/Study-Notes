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
用于给形状 边框增加的样式刷，可以通过$实例化，可复用。

```
var violetbrush = $(go.Brush, "Linear", { 0.0: "Violet", 1.0: "Lavender" });
// 第二个参数   Brush.Solid, Brush.Linear, Brush.Radial, Brush.Pattern,
$(go.Node, "Auto",
      $(go.Shape, "RoundedRectangle",
        { fill: violetbrush }),
      $(go.TextBlock, "Hello!",
        { margin: 5 })
    )
```

笔刷和几何对象可以共享，但GraphObject可以不共享。

##### 模型和模板
模板： 可以为node link 指定特殊的模板，通过Diagram.nodeTemplate
模型： 指定生成图的数据接口， GraphLinksModel就是一类模型， 比如指定nodeDataArray linkDataArray, 最简单的还有树模型。

节点数据对象通常在“key”属性中具有其节点的唯一键值。但是不是一定要用key
```
diagram.nodeTemplate =  // 自定义模板
    $(go.Node, "Auto",
      $(go.Shape,
        { figure: "RoundedRectangle",
          fill: "white" }),
      $(go.TextBlock,
        { text: "hello!",
          margin: 5 })
    );

// 模型初始化
  var nodeDataArray = [
    { key: "Alpha" },
    { key: "Beta" }
  ];
  var linkDataArray = [
    { from: "Alpha", to: "Beta" }
  ];
  diagram.model = new go.GraphLinksModel(nodeDataArray, linkDataArray);
```

修改模型
如果要以编程方式添加或删除节点，则可能需要调用 Model.addNodeData和Model.removeNodeData方法。
如果只有唯一的键值，请使用Model.findNodeDataForKey方法查找特定的节点数据对象。您也可以调用Model.copyNodeData来制作节点数据对象的副本，然后可以对其进行修改并传递给Model.addNodeData。

```
 var data = myDiagram.model.findNodeDataForKey("Delta");
    // 通过setData去驱动数据修改页面结构
    if (data !== null) myDiagram.model.setDataProperty(data, "color", "red");
```

保存和加载模型

只需调用Model.toJson即可生成表示您的模型的字符串。给定由Model.toJson生成的字符串，调用静态方法Model.fromJson来构造和初始化模型。

数据绑定对节点进行参数化 重要 （实现文字输入，实现初始化位置）
go.Binding 用于给模板中某个属性绑定指定的数据，第三个参数可以是一个转换函数
```
diagram.nodeTemplate =
    $(go.Node, "Auto",
      $(go.Shape,
        { figure: "RoundedRectangle",
          fill: "white" },  // default Shape.fill value
        new go.Binding("fill", "color")),  // binding to get fill from nodedata.color
      $(go.TextBlock,
        { margin: 5 },
        new go.Binding("text", "key"))  
        // binding to get TextBlock.text from nodedata.key， 可指定第三个参数类似 new go.Binding("text", "say", function(v) { return "I say: " + v; }) 用于对传入数据进行转换，转换函数不能有任何副作用。转换函数可能会频繁调用，因此它们应该很快并避免分配内存。转换函数的调用顺序没有指定，可能会有所不同。
        // 高级转换，
    );

  var nodeDataArray = [
    { key: "Alpha", color: "lightblue" },  // note extra property for each node data: color
    { key: "Beta", color: "pink" }
  ];
  var linkDataArray = [
    { from: "Alpha", to: "Beta" }
  ];
  diagram.model = new go.GraphLinksModel(nodeDataArray, linkDataArray);
```
转换函数只能返回属性值。您不能返回GraphObjects来替换零件可视树中的对象。如果需要基于绑定数据显示不同的GraphObject，则可以绑定GraphObject.visible或GraphObject.opacity属性。如果您确实想要不同的视觉结构，则可以使用多个模板（Template Maps）。

###### 数据绑定
通常数从源对象到目标对象的设置属性的方法，目标对象通常是GraphObject；源对象通常是模型中保存的JavaScript数据对象。

位置绑定
Part.location的值是Point，因此在此示例中，data属性必须是Point。
```
diagram.nodeTemplate =
    $(go.Node, "Auto",
      new go.Binding("location", "loc"),  // get the Node.location from the data.loc value
      $(go.Shape, "RoundedRectangle",
        { fill: "white" },
        new go.Binding("fill", "color")),
      $(go.TextBlock,
        { margin: 5 },
        new go.Binding("text", "key"))
    );

  var nodeDataArray = [
    // for each node specify the location using Point values
    { key: "Alpha", color: "lightblue", loc: new go.Point(0, 0) },
    { key: "Beta", color: "pink", loc: new go.Point(100, 50) }
  ];
  var linkDataArray = [
    { from: "Alpha", to: "Beta" }
  ];
  diagram.model = new go.GraphLinksModel(nodeDataArray, linkDataArray);
```
为方便实用，传入数据是 ’ 0 40‘这样类型的点，需要提供一个转换函数。

```
 new go.Binding("location", "loc", go.Point.parse),  // 将字符串转换成点
```


单向和双向绑定
默认情况绑定是单向的，调用Panel.updateTargetBindings（重新计算此面板上的所有数据绑定，以便基于此对象的数据属性值将新的属性值分配给此可视化树中的graphobject。）或者Model.setDataProperty进行更新。

```
 diagram.nodeTemplate =
    $(go.Node, "Auto",
      { locationSpot: go.Spot.Center },
      $(go.Shape, "RoundedRectangle",
        { // default values if the data.highlight is undefined:
          fill: "yellow", stroke: "orange", strokeWidth: 2 },
        new go.Binding("fill", "highlight", function(v) { return v ? "pink" : "lightblue"; }),
        new go.Binding("stroke", "highlight", function(v) { return v ? "red" : "blue"; }),
        new go.Binding("strokeWidth", "highlight", function(v) { return v ? 3 : 1; })),
      $(go.TextBlock,
        { margin: 5 },
        new go.Binding("text", "key"))
    );

  diagram.model.nodeDataArray = [
    { key: "Alpha", highlight: false }  // just one node, and no links
  ];

  function flash() {
    // all model changes should happen in a transaction
    diagram.model.commit(function(m) {
      var data = m.nodeDataArray[0];  // get the first node data
      m.set(data, "highlight", !data.highlight);
    }, "flash");
  }
  function loop() {
    setTimeout(function() { flash(); loop(); }, 500);
  }
  loop();
```
##### 绑定数据到link
修改图形结构等数据模型数据， 需要使用特定的方法。

对于节点数据，模型方法是 Model.setCategoryForNodeData， Model.setKeyForNodeData， GraphLinksModel.setGroupKeyForNodeData， TreeModel.setParentKeyForNodeData和 TreeModel.setParentLinkCategoryForNodeData。对于链接数据，模型方法为 GraphLinksModel.setCategoryForLinkData， GraphLinksModel.setFromKeyForLinkData， GraphLinksModel.setFromPortIdForLinkData， GraphLinksModel.setToKeyForLinkData， GraphLinksModel.setToPortIdForLinkData和 GraphLinksModel.setLabelKeysForLinkData。

```
function switchTo() {
    // all model changes should happen in a transaction
    diagram.model.commit(function(m) {
      var data = m.linkDataArray[0];  // get the first link data
      if (m.getToKeyForLinkData(data) === "Beta")
        m.setToKeyForLinkData(data, "Gamma");
      else
        m.setToKeyForLinkData(data, "Beta");
    }, "reconnect link");
  }
```
#####绑定数据到graphobject
```
diagram.nodeTemplate =
    $(go.Node, "Auto",
      { selectionAdorned: false },  // no blue selection handle!
      $(go.Shape, "RoundedRectangle",
        // bind Shape.fill to Node.isSelected converted to a color
        new go.Binding("fill", "isSelected", function(sel) {
              return sel ? "dodgerblue" : "lightgray";
            }).ofObject()),  // ofObject意思是名为什么的对象，缺省代表全员
      $(go.TextBlock,
        { margin: 5 },
        new go.Binding("text", "descr"))
    );
 ```   
 
#####绑定数据到模型
 可以绑定数据到模型 方便使用
 ```
   new go.Binding("fill", "color").ofModel()),  // meaning a property of Model.modelData
 
 diagram.model.modelData.color = "yellow";
 ```
 
##### 双向绑定

```
new go.Binding("location", "loc").makeTwoWay(),  // TwoWay Binding
``` 
正如从源到目标时可以使用转换函数一样，可以为Binding.makeTwoWay提供从目标到源的转换函数。例如，在模型数据中以字符串而不是Point表示位置：

// storage representation of Points/Sizes/Rects/Margins/Spots is as strings, not objects:
new go.Binding("location", "loc", go.Point.parse).makeTwoWay(go.Point.stringify)
 
###### 节点数据

对于相同的节点数据，在视图中也会保存两个节点。
所以无法通过数据查询节点。
尽管有Model.findNodeDataForKey方法，但Model上没有这样的方法。但是，如果您确实要搜索具有特定属性的节点，则可以调用Diagram.findNodesByExample。
```
var nodes = myDiagram.findNodesByExample({ text: "something", count: 17 });
  nodes.each(function(n) { console.log(n.key); });
```

