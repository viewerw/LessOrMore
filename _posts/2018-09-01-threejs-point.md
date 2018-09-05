---
layout: post
title: threejs+tweenjs实现3D粒子模型切换
date: 2018-09-01 09:08:00 +0800
categories: threejs webgl
tag: 教程
---

-   content
    {:toc}

## 前言

之前逛论坛时看到一篇利用 three.js 实现粒子模型切换动画的分享，具体的效果如下：

![](/styles/images/three-point1.gif)

也可以去[预览](http://up.qq.com/act/a20170301pre/index.html)。

但是作者并没有把源码分享出来，正好最近在学习 threejs，正好抽时间写了一个类似的 demo，希望能帮助一些喜欢 threejs 的初学者。效果如下：

![](/styles/images/three-point2.gif)

接下来，我们一起来看如何实现这样一个粒子体系切换动画。

## 获取模型

要实现一个 3D 动画的第一步就是设计出场景中的 3D 模型，而我们做 3D 粒子模型切换动画，则还需要将 3D 模型变换成我们需要的粒子模型，然而，我只是一个前端开发，并不会建模，所以只好“窃取”别人的劳动成果了，去[宣传页](http://up.qq.com/act/a20170301pre/index.html)，打开调试面板-网络-XHR，点击 f5 刷新页面，得到我们需要的 3D 粒子模型：

![](/styles/images/json.png)

将获取到的 json 文件保存到本地。

## 实现思路

### threejs 初始化工作

首先，初始化 threejs 三大元素：场景，相机，渲染器。我们需要一个用于切换的载体粒子体系和多个环境粒子体系（为了简单，我只初始化了一个上下转动的环境粒子体系）。载体粒子体系的粒子数量要比所有模型的顶点数量的最大值还要大，这样才能保证切换到每一个模型，都不会出现缺失的情况，而多余的点呢就让他们从头开始重叠好了。

初始化代码：

```
// renderer 的承载容器
container = document.createElement('div');
document.body.appendChild(container);
// 初始化相机
camera = new THREE.PerspectiveCamera(105, window.innerWidth / window.innerHeight, 10, 10000);
camera.position.z = 100;
// 初始化场景
scene = new THREE.Scene();
scene.fog = new THREE.FogExp2(0x000000, 0.001);// 雾化
//初始化renderer
renderer = new THREE.WebGLRenderer();
renderer.setPixelRatio(window.devicePixelRatio);
renderer.setSize(window.innerWidth, window.innerHeight);
container.appendChild(renderer.domElement);
// 初始化geometry
geometry = new THREE.Geometry();
around = new THREE.Geometry();
// 初始化贴图
var textureLoader = new THREE.TextureLoader();
textureLoader.crossOrigin = '';
var mapDot = textureLoader.load('textures/gradient.png');  // 圆点
```

初始化载体粒子体系：

```
//初始变换点组
for (let i = 0; i < 10000; i++) {

    var vertex = new THREE.Vector3();
    vertex.x = 800 * Math.random() - 400;
    vertex.y = 800 * Math.random() - 400;
    vertex.z = 800 * Math.random() - 400;

    geometry.vertices.push(vertex);

    geometry.colors.push(new THREE.Color(1, 1, 1));

}
material = new THREE.PointsMaterial({ size: 4, sizeAttenuation: true, color: 0xffffff, transparent: true, opacity: 1, map: mapDot });

material.vertexColors = THREE.VertexColors;
particles = new THREE.Points(geometry, material);
scene.add(particles);
```

将获取到的 3D 模型，通过 JSONLoader 加载后，得到的 geometry 对象放入一个数组 glist 中，用于模型切换：

加载模型 loadObject：

```
var loader = new THREE.JSONLoader();
loader.load('obj/game.js', function (geo, materials) {
    var colors = [];
    for (var i = 0; i < geo.vertices.length; i++) {
        colors.push(new THREE.Color("rgb(255, 255, 255)"))
    }

    geo.colors = colors;

    //调整geometry在场景中的位置和大小

    geo.center();
    geo.normalize();
    geo.scale(500, 500, 500)
    geo.rotateX(Math.PI / 4)
    geo.rotateY(-Math.PI / 8)
    glist.push(geo)
})
```

### 添加页面事件监听

```
//事件监听
document.addEventListener('mousedown', onDocumentMouseDown, false);
document.addEventListener("mousewheel", onDocumentMouseWheel, false);
document.addEventListener("keydown", onDocumentKeyDown, false);
window.addEventListener('resize', onWindowResize, false);
```

事件监听方面，我做了以下处理：

1. 按住鼠标拖动，可以旋转场景中的粒子体系。
2. 滚动鼠标滚轮，可以拉近或拉远相机。
3. 当按下键盘上方向键上下的时候，展示粒子模型切换动画。

效果：

![](/styles/images/three-point3.gif)

根据鼠标拖动的偏移量决定模型的旋转角度，代码：

```
function onDocumentMouseMove(event) {
    geometry.rotateY((event.pageX - mouseX) / 1000 * 2 * Math.PI);
    geometry.rotateX((event.pageY - mouseY) / 500 * 2 * Math.PI);

    event.preventDefault();
    mouseX = event.pageX;
    mouseY = event.pageY;
}
```

根据滚轮的滚动量决定相机的 z 轴高度，实现缩放，代码：

```
function onDocumentMouseWheel() {
    camera.position.z += event.deltaY;
}
```

判断键盘按键 key 值，决定是否渲染补间动画，代码：

```
function onDocumentKeyDown(event) {
    if (event.which == 40 && objIndex < 4) {
        objIndex++;
        tweenObj(objIndex);
        flag = true;
    } else if (event.which == 38 && objIndex > 0) {
        objIndex--;
        tweenObj(objIndex);
        flag = true;
    }
}
```

### 使用 tweenjs 展示补间动画

```
function tweenObj(index) {
    geometry.vertices.forEach(function (e, i, arr) {
        var length = glist[index].vertices.length;
        var o = glist[index].vertices[i % length];
        new TWEEN.Tween(e).to({
            x: o.x,
            y: o.y,
            z: o.z
        }, 1000).easing(TWEEN.Easing.Exponential.In).delay(1000 * Math.random()).start();
    })
    camera.position.z = 750;
}
```

delay 一个 1000ms 以内的随机数，为了使动画更加平滑。

### 渲染

这是最关键的一步，也是整个场景能够动起来的原因，代码：

```
function render() {
    //初始粒子体系绕Y轴匀速转动
    if (!flag) {
        geometry.rotateY(Math.PI / 200)
    }
    //环境粒子转动
    around.rotateX(Math.PI / 1000)
    //tween 实时更新粒子位置
    TWEEN.update();
    // 指定相机角度
    camera.lookAt(scene.position);
    // 随机变换顶点颜色
    geometry.colors.forEach(function (color) {
        color.setRGB(Math.random() * 1, Math.random() * 1, Math.random() * 1);
    });
    // 设置几何体的顶点和颜色可以被更新
    geometry.verticesNeedUpdate = true;
    geometry.colorsNeedUpdate = true;
    // 渲染器渲染
    renderer.render(scene, camera);
}
```

TWEEN.update()和 geometry.verticesNeedUpdate = true 共同决定了粒子体系切换动画可以展示出来。

## 结语

虽然日常的前端业务开发很少用到 threejs，但是随着 webGL 和硬件设备的发展，相信以后 threejs 会在 webVR 领域大发异彩，让我们一起期待。

欢迎关注，后续还会有 threejs 和微信小程序方面的分享 ^^！
