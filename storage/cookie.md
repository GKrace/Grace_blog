---
layout: http
title: 浏览器存储方式(一) Cookie
date: 2019-12-20 17:00:15
tags: http
---

浏览器本地存储的方式总共有以下几种方式：cookie localStorage sessionStorage

### cookie

cookie 就是一种浏览器管理状态的一个文件，其属性有 name、value 等等。  
cookie 一般被保存在客户端中，按照客户端中存储的位置，可以分为内存 cookie 和硬盘 cookie。

<!-- more -->

- 内存 cookie （会话期 cookie）： 内存 cookie 由浏览器进行维护，保存在内存中，浏览器关闭消失。会话期 cookies 将会在客户端关闭时被移除。 会话期 cookie 不设置 Expires 或 Max-Age 指令。注意浏览器通常支持会话恢复功能。

```
Set-Cookie: sessionid=38afes7a8; HttpOnly; Path=/
```

- 硬盘 cookie （持久化 cookie）： 硬盘 cookie 保存在硬盘里，有一个过期时间，除非用户手动清理，或者到了过期时间，否则不会被删除。持久化 Cookie 不会在客户端关闭时失效，而是在特定的日期（Expires）或者经过一段特定的时间之后（Max-Age）才会失效。

```
Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT; Secure; HttpOnly
```

#### 1. cookie 为什么出现？

http 协议是[无状态的](https://www.zhihu.com/question/23202402),意思是每次请求，服务器并不清楚到底是不是同一个浏览器在访问他，服务器也不清楚用户上一次到底干了什么，cookie 就是用来绕开 http 无状态性的额外手段之一。

#### 2. cookie 是如何工作的

    cookie包含一些数据：实际的value（键值对形式），到期日期（过期失效），发送到服务器的域和路径，当我们向服务器发送含有cookie的页面时，该cookie会被添加到http头部，服务器端读取信息，并确定我们有权查看页面。

#### 3. cookie 的应用

举个例子，在购物网页中，用户选购了一项商品，服务器在向用户发送网页的同时，还发送了一段 cookie，记录了那个商品信息，用户访问另一页面的时候，浏览器会把之前的 cookie 发送给服务器，这样，服务器还知道用户之前选购了什么，结账时，服务器读取发送过来的 cookie 就行了。

Cookie 另一个典型的应用是当登录一个网站时，网站往往会请求用户输入用户名和密码，并且用户可以勾选“下次自动登录”。如果勾选了，那么下次访问同一网站时，用户会发现没输入用户名和密码就已经登录了。这正是因为前一次登录时，服务器发送了包含登录凭据（用户名加密码的某种加密形式）的 Cookie 到用户的硬盘上。第二次登录时，如果该 Cookie 尚未到期，浏览器会发送该 Cookie，服务器验证凭据，于是不必输入用户名和密码就让用户登录了。

#### 4. cookie 的缺陷

- 由于在 HTTP 请求中的 Cookie 是明文传递的，所以安全性成问题，除非用 HTTPS。
- Cookie 会被附加在每个 HTTP 请求中，所以无形中增加了流量。
- Cookie 的大小限制在 4KB 左右，对于复杂的存储需求来说是不够用的。

#### 5. cookie 的属性

<img src="/images/storage_cookie.png" width="100%" height="100%">
如此图显示，cookie主要有以下几个属性。在设置这些属性时，属性之间由一个分号和一个空格隔开。

```
"key=name; expires=Sat Dec 21 2019 21:16:10 GMT; domain=XXXX.com; path=/; secure; HttpOnly"
```

- **name：** cookie 名，一个域名下绑定的 cookie，name 不能相同，相同的 name 值会被覆盖掉。
- **value：** cookie 值，由于 cookie 规定是名称/值是不允许包含分号，逗号，空格的，所以为了不给用户到来麻烦，考虑服务器的兼容性，任何存储 cookie 的数据都应该被编码。
- **Domain=`<domain-value>`：** 指定 cookie 可以送达的主机名。假如没有指定，那么默认值为当前文档访问地址中的主机部分（但是不包含子域名）。与之前的规范不同的是，域名之前的点号会被忽略。假如指定了域名，那么相当于各个子域名也包含在内了。
- **path：** 指定一个 URL 路径，这个路径必须出现在要请求的资源的路径中才可以发送 Cookie 首部。字符 %x2F ("/") 可以解释为文件目录分隔符，此目录的下级目录也满足匹配的条件（例如，如果 path=/docs，那么 "/docs", "/docs/Web/" 或者 "/docs/Web/HTTP" 都满足匹配的条件）。
- **Expires=`<date>`：** cookie 的最长有效时间，如果没有设置这个属性，则表示是一个会话期 cookie，一个会话结束于客户端被关闭，则会话期 cookie 会被删除，现在有些浏览器支持恢复会话恢复功能，即在浏览器关闭时保留所有的 tab 标签，重新打开浏览器的时候会被还原，同时 cookie 也会被恢复。
- **Max-Age=`<non-zero-digit>`：** 在 cookie 失效之前需要经过的秒数。秒数为 0 或 -1 将会使 cookie 直接过期。一些老的浏览器（ie6、ie7 和 ie8）不支持这个属性。对于其他浏览器来说，假如二者 （指 Expires 和 Max-Age） 均存在，那么 Max-Age 优先级更高。
  > 两者区别：expires 是 http/1.0 协议中的选项，在新的 http/1.1 协议中 expires 已经由 max-age 选项代替，两者的作用都是限制 cookie 的有效时间。expires 的值是一个时间点（cookie 失效时刻= expires），而 max-age 的值是一个以秒为单位时间段（cookie 失效时刻= 创建时刻+ max-age）。
  > 另外，max-age 的默认值是 -1(即有效期为 session )；若 max-age 有三种可能值：负数、0、正数。负数：有效期 session；0：删除 cookie；正数：有效期为创建时刻+ max-age
- **secure：** 一个带有安全属性的 cookie 只有在请求使用 SSL 和 HTTPS 协议的时候才会被发送到服务器。然而，保密或敏感信息永远不要在 HTTP cookie 中存储或传输，因为整个机制从本质上来说都是不安全的，比如前述协议并不意味着所有的信息都是经过加密的。  
  <img src="/images/storage_secure.png" width="50%" height="60%">

- **HttpOnly：** 如果这个属性设置为 true，此 cookie 不能使用 JavaScript 经由 Document.cookie 属性、XMLHttpRequest 和 Request APIs 进行访问，以防范跨站脚本攻击（XSS）。

#### 6. cookie 的作用域

Domain 和 Path 标识定义了 Cookie 的作用域：即 Cookie 应该发送给哪些 URL。domain 是域名，path 是路径，两者加起来就构成了 URL，domain 和 path 一起来限制 cookie 能被哪些 URL 访问。

- Domain 标识指定了哪些主机可以接受 Cookie。如果不指定，默认为当前文档的主机（不包含子域名）。如果指定了 Domain，则一般包含子域名。例如，如果设置 Domain=mozilla.org，则 Cookie 也包含在子域名中（如 developer.mozilla.org）。
- Path 标识指定了主机下的哪些路径可以接受 Cookie（该 URL 路径必须存在于请求 URL 中）。以字符 %x2F ("/") 作为路径分隔符，子路径也会被匹配。
  例如，设置 Path=/docs，则以下地址都会匹配：
  > /docs
  > /docs/Web/
  > /docs/Web/HTTP

#### 7. SameSite Cookies

sameSite cookie 允许服务器要求你某个 cookie 在跨站请求时不会被发送，从而可以阻止跨站请求伪造攻击（CSRF）举例如下：

```
Set-Cookie: key=value; SameSite=Strict

// none：浏览器会在同站请求、跨站请求下继续发送cookies，不区分大小写
// script：浏览器将只发送相同站点请求的cookie(即当前网页URL与请求目标URL完全一致)。如果请求来自与当前location的URL不同的URL，则不包括标记为Strict属性的cookie。
// lax：在新版本浏览器中，为默认选项，Same-site cookies 将会为一些跨站子请求保留，如图片加载或者frames的调用，但只有当用户从外部站点导航到URL时才会发送。如link链接

```

> 以前，如果 SameSite 属性没有设置，或者没有得到运行浏览器的支持，那么它的行为等同于 None，Cookies 会被包含在任何请求中——包括跨站请求。但是，在新版本的浏览器中，SameSite 的默认属性是 SameSite=Lax。换句话说，当 Cookie 没有设置 SameSite 属性时，将会视作 SameSite 属性被设置为 Lax——这意味着 Cookies 将会在当前用户使用时被自动发送。如果想要指定 Cookies 在同站、跨站请求都被发送，那么需要明确指定 SameSite 为 None

#### 8. JavaScript 如何操作 cookie ？

**`1. document.cookie 创建新的 cookie`**
通过 Document.cookie 属性可创建新的 Cookie，只可通过该属性访问非 HttpOnly 标记的 Cookie。此方法一次只能对一个 coookie 进行设置或更新。打印出的结果是一个字符串类型，因为 cookie 本身就是存储在浏览器中的字符串。但这个字符串是有格式的，由键值对`key=value` 构成，键值对之间由一个分号和一个空格隔开。

```
document.cookie = "yummy_cookie=choco";
document.cookie = "tasty_cookie=strawberry";
console.log(document.cookie);
```

**`2. 删除 cookie 修改 cookie`**
name/domain/path 这 3 个字段都相同的时候,cookie 会被覆盖
修改 cookie 只需要重新赋值，属性 path/domain 等要和旧 cookie 保持一致，否则会重新创建新的 cookie
删除 cookie 也需要重新赋值，把 cookie 的 expires 设置成一个过去的时间点，同样注意其各种属性一定要旧 cookie 保持一样。

#### 9. 服务端如何设置 cookie

服务端就是通过 setCookie 来设置 cookie 的，设置多个 cookie 时，得多写几个 setCookie
