---
layout: post
title: canvas实现画中画动画效果--网易娱乐年度盘点H5动画解密
date: 2018-11-05 09:08:00 +0800
categories: H5
tag: 教程
---

-   content
    {:toc}

## 前言

由于本人最近在做一些 growth hacking 的工作，业务上以后可能也会涉及去做一些能够在朋友圈火爆分享的 H5 页面，突然想到去年看到一个网易娱乐年度新闻盘点的 H5 页面非常的新颖，采用画中画的形式依次串联十多个手绘娱乐图片，加上洗脑的“好运来”音乐，让人有很大的分享的欲望。

手机扫码体验网易年度娱乐盘点：

![](/styles/images/pip-net.png)

## 一步步实现

接下来我们来一步步实现这样的一个 H5 页面，首先，我们需要搞懂这个页面用到了那些前端的知识点。

### css3 动画 animation

首屏有很多动画，其中大多数是用雪碧图+animation 的 step 动画函数实现的，包括底部的鼓，右上角的镲，中间人物飘动的头发。脚下来回滚动的浪花就是普通的 animation 动画。除了首屏的这些动画，后面切换到某些场景的时候也会有动画，这些动画也是用的雪碧图动画。

### 背景音乐

这个就是使用 audio 元素即可，设置 audio 为循环播放，当点击右上角镲的动图的时候，调用 audio.stop()即可。

### 场景切换

在首屏，当长按鼓的时候，页面的 animation 动画会停止，静态画面一点点的缩小，直至出现第一个完整的画中画。此时过渡动画停止，页面 animation 动画（白百何一指禅）开始出现。

我们先来分析这一小段，我们代码上要做哪些工作。

首先，我们需要两个图层，一个 canvas 图层用来展示场景过渡动画，z-index 较低；一个展示场景动画的图层，我们叫做 gif 图层，z-index 较高；

#### drawImage

在 canvas 图层里，我们使用 drawImage()这个方法来绘制每一帧的过渡图片，我们先来看看这个方法的使用方式：

> context.drawImage(img,sx,sy,swidth,sheight,x,y,width,height);

参数值

| 参数    |                     描述                      |
| ------- | :-------------------------------------------: |
| img     |        规定要使用的图像、画布或视频。         |
| sx      |         可选。开始剪切的 x 坐标位置。         |
| sy      |         可选。开始剪切的 y 坐标位置。         |
| swidth  |           可选。被剪切图像的宽度。            |
| sHeight |           可选。被剪切图像的高度。            |
| x       |        在画布上放置图像的 x 坐标位置。        |
| y       |        在画布上放置图像的 y 坐标位置。        |
| width   | 可选。要使用的图像的宽度。（伸展或缩小图像）  |
| height  | 可选。要使用的图像的高度。（伸展或缩小图像)。 |

过渡动画的每一帧，我们都要在 canvas 上面使用 drawImage 绘制两张图片，一张是大图，一张是画中画里的小图，以第一个过渡动画为例，大图是 P2,小图是 P1,

如图：

![](/styles/images/pip-analysis.jpeg)

我们假设大图 P2 是长方形 ABCD，小图 P1 是长方形 IJKL，动画过程中某一时刻的手机屏幕是长方形 EFGH，我们有个前提条件就是这三个长方形都是宽高比为 750：1206 的长方形，而且，所有的图片宽高像素大小是相等的（网易的场景图片大小统一为：1875\*3015）这也意味着 iPhone X 等全面屏手机的适配会有问题，在 iPhone678 手机上表现良好。（看看今年网易会不会解决这个问题，毕竟全面屏手机越来越多）。

那么，在这样的一个时刻，我们需要在 canvas 上面画两张图片，

    drawImage(P2,ME,NE,EF,EH,0,0,750,1206)
    drwaImage(P1,0,0,AB,AD,OI,PI,IJ,IL)

那我们知道了某一时刻的情况，但是如何将画面动起来，有一个收缩画面的效果呢？

现在开始写我们的 render 函数：

    const render = () => {
        this.radio = this.radio * this.scale;
        this.timer = requestAnimationFrame(render);
        this.draw();// 绘制两个图片
    };

    draw() {
        if (this.index + 1 != this.imgList.length) {
            if (
                this.radio <
                this.imgList[this.index + 1].areaW / this.imgList[this.index + 1].imgW
            ) {
                if (this.willPause) {
                    this.radio =
                        this.imgList[this.index + 1].areaW / this.imgList[this.index + 1].imgW;
                    cancelAnimationFrame(this.timer);
                }
                this.index++;
                this.radio = 1;
                if (!this.imgList[this.index + 1]) {
                    this.showEnd();
                }
            }
            this.imgNext = this.imgList[this.index + 1];
            this.imgCur = this.imgList[this.index];
            this.containerImage = this.domList[this.index + 1];
            this.innerImage = this.domList[this.index];
            this.drawImgOversize(
                this.containerImage,
                this.imgNext.imgW,
                this.imgNext.imgH,
                this.imgNext.areaW,
                this.imgNext.areaH,
                this.imgNext.areaL,
                this.imgNext.areaT,
                this.radio,
            ),
                this.drawImgMinisize(
                    this.innerImage,
                    this.imgCur.imgW,
                    this.imgCur.imgH,
                    this.imgNext.imgW,
                    this.imgNext.imgH,
                    this.imgNext.areaW,
                    this.imgNext.areaH,
                    this.imgNext.areaL,
                    this.imgNext.areaT,
                    this.radio,
                );
        }
    }

render 函数里面有两个变量 radio 和 scale，radio = IJ/EF,所以在一个场景切换动画中，我们只需要改变 radio 的值，使其从 1 逐渐变小到等于 IJ/AB 即可。scale 就是这样一个用来表示 radio 变化速率的常量。这里我们可以定义为 0.99，因为 requestAnimationFrame 的回调在浏览器里面大约一秒会执行 60 次，
而 o.99^240 = 0.08 所以大约 4s 左右，我们就可以完成一个场景切换，这个速度还是比较适中的。

从而，在动画中的任一时刻，EF 的大小可以表示为 IJ/this.radio,另外，因为所有的图片都是我们的画师制作的，所以，每张图的像素大小(imgW、imgH)、小图在大图中的偏移位置 SI(areaL)、TI(areaT)、小图的宽高 IJ(areaW)、IL(areaH),都是已知的，根据这些已知的数据，我们可以轻松的（对于数学好的同学）将 drawImage 中未知变量用 this.radio 表示。

这样，我们一个切换动画算是搞定了，但是我们如何将多个切换动画串联起来呢，很简单，看看 draw()的代码，我们只需要在 this.radio 达到临界值时候，将 index++,重新给 imgNext 和 imgCur 赋值。

最后将 render 函数写到 touchHandler 里面即可。

    touchHandler(e) {
        e.stopPropagation();
        // e.preventDefault();
        const render = () => {
            this.radio = this.radio * this.scale;
            this.timer = requestAnimationFrame(render);
            this.draw();
        };
        cancelAnimationFrame(this.timer);
        this.willPause = false;
        // clearInterval(this.gif_timer);
        this.timer = requestAnimationFrame(render);
    }

### gif 动画

说是 gif 动画，但是实现上还是用雪碧图+step 实现的。如果某一场景中有动画展示的环节，那么在过渡动画结束时，我就可以将 gif 图层展示出来，gif 图层有两部分构成，一个是背景图片，一个是动画区域。背景图片将动画区域留白，动画区域采用雪碧图+step 的方式，实现动画。这样做是为了减少图片资源大小，加快加载速度。

### 加载图片

这个 H5 页面需要加载大量的图片，而这些图片一定要保证在用户交互之前加载完成，所以我们要给页面初始化时候一个加载态，当所有图片加载完成后，我们才展示可交互的页面。所以，我们需要知道什么时候图片已经加载好了，上代码：

    loadGifImg() {
        const loadPromises = this.gifImgs.map(
            item =>
                new Promise((resolve, reject) => {
                    const img = new Image();
                    img.src = item;
                    img.onload = () => resolve(img);
                    img.onerror = () => reject();
                }),
        );
        return Promise.all(loadPromises);
    }

    loadPageImg() {
        const loadPromises = this.imgList.map(
            (item, index) =>
                new Promise((resolve, reject) => {
                    const img = new Image();
                    img.src = item.link;
                    img.i = index;
                    img.name = index;
                    img.className = 'item';
                    item.image = img;
                    img.onload = () => {
                        $('.collection').append(item.image);
                        resolve();
                    };
                    img.onerror = () => reject();
                }),
        );
        return Promise.all(loadPromises);
    }

所以，我们只需要等这两个 Promise resolve 了就加载完成了。

## 最后

[完整的代码](https://github.com/viewerw/pipcanvas) 欢迎 star

手机扫码体验

![](/styles/images/pip-preview.png)
