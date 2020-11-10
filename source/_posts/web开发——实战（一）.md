---
title: web开发——实战（一）
date: 2020-05-13 20:30:49
tags: [web, leaflet]
---

## leaflet入门

> 参考文章：https://leafletjs.com/examples/quick-start/

<!-- more -->

### 准备页面

在为地图编写任何代码之前，您需要在页面上执行以下准备步骤：

- 在文档的头部添加Leaflet CSS文件：

  ```javascript
   <link rel="stylesheet" href="https://unpkg.com/leaflet@1.6.0/dist/leaflet.css"
     integrity="sha512-xwE/Az9zrjBIphAcBb3F6JVqxf46+CDLwfLMHloNu6KEQCAWi6HcDUbeOfBIptF7tcCzusKFjFw2yuvEpDL9wQ=="
     crossorigin=""/>
  ```

- **在** Leaflet的CSS **之后**包括Leaflet JavaScript文件：

  ```javascript
   <!-- Make sure you put this AFTER Leaflet's CSS -->
   <script src="https://unpkg.com/leaflet@1.6.0/dist/leaflet.js"
     integrity="sha512-gZwIG9x3wUXg2hdXF6+rVkLF/0Vi9U8D2Ntg4Ga5I5BZpVkVxlJWbSQtXPSiUTtC0TjtGOmxa1AJPuV0CPthew=="
     crossorigin=""></script>
  ```

- 在要放置地图的位置放一个`div`元素`id`：

  ```javascript
   <div id="mapid"></div>
  ```

- 确保地图容器具有定义的高度，例如通过在CSS中进行设置：

  ```javascript
  #mapid { height: 180px; }
  ```

现在，您可以初始化地图并对其进行一些处理。

### 标记，圆形和多边形

除了图块图层，您还可以轻松地向地图添加其他内容，包括标记，折线，多边形，圆形和弹出窗口。让我们添加一个标记：

```javascript
var marker = L.marker([51.5, -0.09]).addTo(mymap);
```

添加圆是一样的（除了以米为单位指定半径作为第二个参数），但是您可以通过在创建对象时将选项作为最后一个参数传递来控制圆的外观：

```js
var circle = L.circle([51.508, -0.11], {
    color: 'red',
    fillColor: '#f03',
    fillOpacity: 0.5,
    radius: 500
}).addTo(mymap);
```

添加多边形非常简单：

```javascript
var polygon = L.polygon([
    [51.509, -0.08],
    [51.503, -0.06],
    [51.51, -0.047]
]).addTo(mymap);
```

### 使用弹出窗口

当您要将某些信息附加到地图上的特定对象时，通常使用弹出窗口。Leaflet有一个非常方便的快捷方式：

```javascript
marker.bindPopup("<b>Hello world!</b><br>I am a popup.").openPopup();
circle.bindPopup("I am a circle.");
polygon.bindPopup("I am a polygon.");
```

尝试点击我们的对象。该`bindPopup`方法将带有指定HTML内容的弹出窗口附加到您的标记，因此当您单击对象时弹出窗口会出现，并且该`openPopup`方法（仅用于标记）会立即打开附加的弹出窗口。

您还可以将弹出窗口用作图层（当您需要的不仅仅是将弹出窗口附加到对象时）：

```javascript
var popup = L.popup()
    .setLatLng([51.5, -0.09])
    .setContent("I am a standalone popup.")
    .openOn(mymap);
```

在这里，我们使用`openOn`而不是`addTo`因为它在打开一个对可用性有利的新弹出窗口时会自动关闭先前打开的弹出窗口。

### 处理事件

每当Leaflet中发生任何事情时，例如用户单击标记或更改地图缩放，相应的对象都会发送一个事件，您可以使用功能订阅该事件。它允许您对用户交互做出反应：

```javascript
function onMapClick(e) {
    alert("You clicked the map at " + e.latlng);
}

mymap.on('click', onMapClick);
```

每个对象都有自己的事件集- 有关详细信息，请参见[文档](https://leafletjs.com/reference.html)。侦听器函数的第一个参数是事件对象-它包含有关发生的事件的有用信息。例如，地图单击事件对象（`e`在上面的示例中）具有`latlng`属性，该属性是单击发生的位置。

让我们通过使用弹出窗口而不是警报来改进示例：

```javascript
var popup = L.popup();

function onMapClick(e) {
    popup
        .setLatLng(e.latlng)
        .setContent("You clicked the map at " + e.latlng.toString())
        .openOn(mymap);
}

mymap.on('click', onMapClick);
```

尝试单击地图，您将在弹出窗口中看到坐标。



## D3.js 入门

**D3.js**是一个JavaScript库，用于根据数据处理文档。**D3**可帮助您使用HTML，SVG和CSS使数据栩栩如生。D3对Web标准的重视为您提供了现代浏览器的全部功能，而又不会使自己陷入专有框架，而是结合了强大的可视化组件和数据驱动的DOM操作方法。