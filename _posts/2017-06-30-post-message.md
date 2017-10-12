---
layout: post
title:  postMessage使用以及demo
date:   2017-06-30 09:08:00 +0800
categories: Html5
tag: HTML5、javascript
---

* content 
{:toc}


前言
===================

  因为业务需求需要用到多窗口之间通信，想起之前听说过postMessage可以实现相关功能，就研究了一下，并写了个小demo。

postMessage介绍
===================

> otherWindow.postMessage(message,targetOrigin)

### otherWindow

这是一个对其他窗口的引用，可以通过以下方式获取：
1. 通过一个iframe对象的contentWindow属性获取；
2. window.open()的返回值；
3. 通过window.frames[arg]，其中arg是窗口名称或者index值。
4. 如果，你想从iframe中向父窗口传递信息，那么parent也是一个合理的引用。

注：这个window对象表示，消息进行dispatch的起始窗口，如果，你想通过父window向子window传递消息，那么otherWindow应该是window.frames[0]，如果，子窗口想向父窗口传递消息，那么otherWindow应该是window.parent。

### message

这是通过postMessage传递的消息对象。它有三个属性 ：

#### data

传递的消息，值一般是字符串，可以是json解析的字符串。

#### orgin

消息发送的源地址，格式：协议+主机+端口

#### source

消息发送的源窗口引用。

### targetOrigin

指定otherWindow的源（协议+主机+端口），来决定消息是否应该被分发。如果，otherWindow的源和targetOrigin不匹配，那么消息将停止在该窗口分发。当然，我们也可以指定成通配符'*',表示不需要校验源。为了安全起见，防止信息泄露，最好指定源。

### 消息接受

通过以下JavaScript代码实现消息的接受：
	window.addEventListener("message", receiveMessage, false);

	function receiveMessage(event)
	{
	  if (event.origin !== "http://example.org:8080")
	    return;

	  // ...
	}

其他
===============

如果你不想接收消息，最安全的做法，就是不写对message事件的监听回调

demo
===========================

### a.html

	<!DOCTYPE html>
	<html>
	<head>
		<title></title>
		<script type="text/javascript">
			window.addEventListener('message',function(e){
	                //if(e.source!=window.frames[0]) return;
	                
	                alert(e.data)
	            },false);
		</script>
	</head>
	<body>
	<iframe src="b.html"></iframe>
	</body>
	</html>

### b.html

	<!DOCTYPE html>
	<html>
	<head>
		<title></title>
		<script type="text/javascript">
			function postM(){
				window.parent.postMessage('aaa','*')
			}
		</script>
	</head>

	<body>
	<button onclick="postM()">post</button>
	</body>
	</html>
 
在浏览器打开a.html，点击post按钮，你会看到alert框。