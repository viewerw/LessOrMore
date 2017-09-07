---
layout: post
title:  vue 爬坑之路（持续更新）
date:   2017-08-30 09:08:00 +0800
categories: vue
tag: 教程
---

* content
{:toc}


前言
===========================
<hr>
我使用vue进行开发前端页面已经半年多时间了，从最开始只看官方文档，自己写demo，到后来接触公司vue项目，渐渐成为一名重度vue使用依赖者，几乎现在的开发已经不使用jQuery了。但是，有时维护公司的老项目，又要去看那些代码，感觉像是看一件件古董，它们虽然写的也很完美，很精致，细细读完，也能感叹`造物者`精妙的设计。但是，对于公司不断扩张的需求和周期很短的项目开发，这种老的开发模式就像古董一样没有了使用价值。在单页面应用于组件化开发横行的今天，vue的使用，让我在代码开发过程中感到了如鱼得水。本篇主要记录，vue新手在开发过程中可能会遇到的一些奇异的百思不得其解的问题，这些其实都跟vue的一些机制有关，让我们一起走进它们。

data中 对象的属性附加问题
==========================
<hr>

场景
--------------------------
<hr>
实际项目开发过程中会经常遇到，我们从后台接口获取一些对象组成的list，然后在模板里面通过`v-for`渲染这些数据，然而，我们不仅仅要展示这些数据，我们可能需要对每一项有一些交互动作，从而改变对象数据的某些状态，这些状态属性可能接口并没有定义返回给我们，那就需要我们自己给数组中的对象添加一些额外的属性。

例子
-------------------------------
<hr>
![云盘图片](/styles/images/yupan.png)
以最近开发的云盘的项目为例，从图中可以看到，每个文件都需要一个状态来表示当前是否是处于被用户选中的状态，然而，后台查询接口返回的`fileList`里每个`file`对象里面是肯定没有这个属性的，因为这是前台的交互行为附加的属性。那么，我们怎么去加上这个属性呢？如果，想法是用户点击这个文件的时候，给这个文件对象加上附加属性。代码可能是这样的：

    chooseFile(item){
		item.choosed?item.choosed=false:item.choosed=true;
    },

这样的问题是什么呢，就是点击文件并不会出现选中的小勾，而这时，那个点击的文件对象的choosed属性值也确实是true，而这边这样写的样式确没有加载出来

	<div  :class = "{checked:item.choosed,check:!item.choosed}"></div>

这时，就有点抓狂了，你会对着`console`里面打印出来的file的choosed属性值 和 页面上并不相称的渲染结果陷入深思。。。难道是vue数据渲染机制出bug了吗？

解决
------------------------------------
首先，我们看看官网上的一段话：
>由于 JavaScript 的限制， Vue 不能检测以下变动的数组：
>
>1. 当你利用索引直接设置一个项时，例如： vm.items[indexOfItem] = newValue
>2. 当你修改数组的长度时，例如： vm.items.length = newLength
>
> 解决第一类问题：Vue.set(example1.items, indexOfItem, newValue)
>
>解决第二类问题：数组整体的重新替换	

因为官网上举例用的是数组，所以，新手给对象的属性赋值就会出现上面的错误，所以，刚刚的代码可以这样写

	chooseFile(item){
		item.choosed?this.$set(item,'choosed',false):this.$set(item,'choosed',true);
    },

那么，你学到了吗？


事件绑定
======================
<hr>

场景
----------------------
<hr>
  这个场景太多了，vue组件向外部传递数据就是通过事件触发的方式，我们通过在外部指定一个`method`来处理组件触发的事件。

例子
------------------------
<hr>
  当用户输入关键字进行搜索的时候，我们不能在每个`input`事件被触发的时候都去调用接口查询，一来，接口返回有延迟，会导致查询结果慢一拍，还不停变化，二来，对接口的请求次数过高，服务器压力大。这时，我们需要用到去抖函数`debounce`。
  查询框模板：

    <input @input=" searchPartner" type="" placeholder="请输入同伴姓名" v-model.trim="search_info">

  查询方法：
	
	methods：{
			searchPartner(){
                let self = this;
                return _.debounce(function(){self.search_stu_info()},400)
            },
            search_stu_info(){
            	...调接口
            }
	}
	
  当我写完代码运行的时候，发现根本就没有调用接口，也就是根本没有走到`search_stu_info`方法，于是，我认为是事件绑定出了问题，input事件绑定的方法根本不是去抖函数的返回值，于是，我修改了模板：

    <input @input=" searchPartner()" type="" placeholder="请输入同伴姓名" v-model.trim="search_info">

  只是加了括号，我希望`searchPartner`自执行，从而返回去抖函数绑定到input事件。再次运行，发现还是没调用接口，想了一会儿，想不通==！

解决
--------------------
<hr>
  当然，这个问题首先需要搞明白的是在vue事件绑定中，`@input=" searchPartner"`和`@input=" searchPartner()"`有什么区别，答案就是没啥区别。。
  上源码：

    if (!handler.modifiers) {
	    return simplePathRE.test(handler.value)
	      ? handler.value
	      : `function($event){${handler.value}}`
	}

  带括号的生成

    on: {
       "click": function($event) {
             doXX()
          }
    }

  不带括号生成：

    on: { "click": doXX }

  所以区别就是外面包裹了一个function，所以带括号的方法也并不会自执行，它只会被当成字符串原封不动的被搬过来，所以，绑定到input事件的方法，永远都不可能是`searchPartner()`的返回值，那该怎么写呢，后来看别人的代码发现是这样写的：

    searchPartner:_.debounce(function(){this.search_stu_info()},400),

  因为用es6语法用的久了，竟然忘记methods是一个对象，`searchPartner(){}`其实是等价于`searchPartner:function(){}`,所以，为什么要把`_.debounce(function(){this.search_stu_info()},400)`写在return 里面呢。

  所以，你学到了吗？

vue的复用机制
=====================
<hr>

场景
----------------------
<hr>

  使用vue写项目，会经常频繁的使用到`v-for`去渲染列表，有一些列表项是不带状态的，有些是带状态的（例如，是否勾选），vue在设计的时候，为了更高效率的渲染出dom，它提供了模板复用的机制。

例子
-----------------------
<hr>

![正常效果](/styles/images/rightkey.gif)
  出于项目需求，我需要实现一个双向选择的穿梭框，勾选左边列表的任意一项，该项都会出现在右侧列表，点击右侧列表的任一项后面的删除图标，左侧列表项前面的勾选状态都要去掉，同时，点击左侧列表每一项后面的`>>`，可以支持进入其子项的列表。例如，首先左侧列表展示所有的班级列表，点击任一项后面的`>>`图标，都可以进入该班级下面的学生列表，学生列表依旧展示在左侧。所有的班级项和学生项，都支持通过勾选移动到右侧。

  初步分析之后，借助elmentui提供的元素，很快写出了初步的实现效果，但是很快发现了一个问题，当我选中班级列表的第一项，然后再进入其它任一班级的学生列表，这时候，发现，学生列表的第一项会被默认选中，而右侧没有这条数据。通过打印log发现，该学生项的选中状态也是 false，所以，第一个学生项为什么会被选中呢？？（wtf？？黑人问号）

  如图：

![错误效果](/styles/images/errkey.gif)

解决
--------------------------------
<hr>

  这个问题就是由于vue的复用机制导致的！

  首先：

  >当 Vue.js 用 v-for 正在更新已渲染过的元素列表时，它默认用 “就地复用” 策略。如果数据项的顺序被改变，Vue将不是移动 DOM 元素来匹配数据项的顺序， 而是简单复用此处每个元素，并且确保它在特定索引下显示已被渲染过的每个元素。

  >这个默认的模式是高效的，但是只适用于不依赖子组件状态或临时 DOM 状态（例如：表单输入值）的列表渲染输出。

  然而，我这个问题中，恰好复用的多选框是依赖 组件的状态的，so，简单的复用原则会把班级列表的第一项的多选框复用到学生列表的第一项的多选框，那么怎么解决呢？

  很简单，给`v-for`渲染的元素加上一个唯一的key值，这样vue的默认复用机制就不生效了，它会重新渲染元素，而不是简单的复用。

  代码如下：

	<div style="height:290px;overflow-y:auto">
        <div v-for="(item,index) in sdata1" class = "check-item" :key="item.id">
           <el-checkbox @change = "itemClick(item)" v-model="item.checked">{{item.name}}</el-checkbox>

           <i v-show="!item.checked&&!item.end" title="进入" style = "width:30px;height:30px;line-height:30px;text-align:center" class = "el-icon-d-arrow-right" @click.stop = "getChildren(item)"></i>
        </div>
   	</div>

  这样做，可能会有一个bug，就是当学生的id和班级的id相同的时候，可能元素又被复用，当然，这样的概率很小。

  所以，你学到了吗？
