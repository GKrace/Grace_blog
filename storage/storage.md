---
layout: http
title: 浏览器存储方式(二) LocalStorage 和 SessionStorage
date: 2019-12-30 17:16:20
tags: http
---

Web Storage 包括了两种存储方式：sessionStorage 和 localStorage。
Web Storage 的目的是克服由 cookie 带来的一些限 制，**当数据需要被严格控制在客户端上时，无须持续地将数据发回服务器**。Web Storage 的两个主要目标是:

<!-- more -->

- 提供一种在 cookie 之外存储会话数据的途径;
- 提供一种存储大量可以跨会话存在的数据的机制。
  最初的 Web Storage 规范包含了两种对象的定义:sessionStorage 和 globalStorage。这两个
  对象在支持的浏览器中都是以 windows 对象属性的形式存在的，支持这两个属性的浏览器包括 IE8+、 Firefox 3.5+、
  Chrome 4+和 Opera 10.5+。

Firefox 2 和 3 基于早期规范的内容部分实现了 Web Storage，当时只实现了 globalStorage，没有实现 localStorage。

### Web Storage 为什么更适合存储大量数据？

- 每个域 Chrome，Firefox 和 Opera 是 5M，IE 是 10M。 可以用这个来测 http://dev-test.nemikor.com/web-storage/support-test/ 。
- 请求时不会带 web stroge 的内容。

曾经，Firefox 支持 globalStorage：能读写所有域的存储数据的 localStorage。但 globalStorage 没有成为标准。Firefox 13.0 后被废弃了。

### storage 对象简介

Storage 类型提供最大的存储空间(因浏览器而异)来存储名值对儿，主要有以下方法：

- clear(): 删除所有值;Firefox 中没有实现 。
- getItem(name):根据指定的名字 name 获取对应的值。
- key(index):获得 index 位置处的值的名字。
- removeItem(name):删除由 name 指定的名值对儿。
- setItem(name, value):为指定的 name 设置一个对应的值。

建议使用方法而不是属 性来访问数据，以免某个键会意外重写该对象上已经存在的成员。Storage 类型只能存储字符串。非字符串的数据在存储之前会被转换成字符串。

### sessionStorage

sessionStorage 对象存储特定于某个会话的数据，关闭浏览器消失。存储在 sessionStorage 中的数据可以跨越页面刷新而 存在，同时如果浏览器支持，浏览器崩溃并重启之后依然可用(Firefox 和 WebKit 都支持，IE 则不行)。
因为 seesionStorage 对象绑定于某个服务器会话，所以当文件在本地运行的时候是不可用的。存储在 sessionStorage 中的数据只能由最初给对象存储数据的页面访问到，所以对多页面应用有限制。 由于 sessionStorage 对象其实是 Storage 的一个实例，所以可以使用 setItem()或者直接设置新的属性来存储数据。下面是这两种方法的例子。

```
//使用方法存储数据
sessionStorage.setItem("name", "Grace");

//使用属性存储数据
sessionStorage.book = 'grace blog';

```

<img src='/images/storage_localStorage.jpg' width="50%" height="50%">

### globalStorage

<p style="color:red;">globalStorage 已经废弃，替代品为 localStorage</p>
Firefox2 中实现了 globalStorage 对象，这个对象的目的是跨会话存储数据，但是有特定的访问限制。要使用 globalStorage，首先要制定哪些域可以访问该数据，可以通过括号标记使用属性来实现

```
//保存数据
globalStorage["wrox.com"].name = "Nicholas";
//获取数据
var name = globalStorage["wrox.com"].name;

```

globalStorage 对象不是 Storage 的实例， 而具体的 `globalStorage["wrox.com"]`才是，

### localStorage

localStorage 方法存储的数据没有时间限制。第二天、第二周甚至是第二年之后，数据依然可用。永久存储除非手动删除 方法简介明了 容易操作。不能跨浏览器读取数据。不能给 localstorage 制定任何访问规则，规则事先就设定好了，要访问同一个 localStorage 对象，页面必须来自同一个域名（子域名无效），使用同一种协议，在同一个端口上，相当于`globalStorge[location.host]`
localstorage 是 storage 的实例，其使用方法同 sessionstorage

### cookie localStorage sessionStorage 的区别

共同点：都是保存在浏览器端，且同源的。
区别：

- 与服务器的数据交换方式不同。 cookie 数据始终在同源的 http 请求中携带（即使不需要），即 cookie 在浏览器和服务器间来回传递。而 sessionStorage 和 localStorage 不会自动把数据发给服务器，仅在本地保存。
- 存储大小限制也不同，cookie 数据不能超过 4k，同时因为每次 http 请求都会携带 cookie，所以 cookie 只适合保存很小的数据，如会话标识。sessionStorage 和 localStorage 虽然也有存储大小的限制，但比 cookie 大得多，可以达到 5M 或更大。
- 数据有效期不同，sessionStorage：仅在当前浏览器窗口关闭前有效，自然也就不可能持久保持；localStorage：始终有效，窗口或浏览器关闭也一直保存，因此用作持久数据；cookie 只在设置的 cookie 过期时间之前一直有效，即使窗口或浏览器关闭。
- Cookie 可以设置有效期，路径(path)，域（domain）
