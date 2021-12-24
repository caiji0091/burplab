---
description: 本章将介绍什么跨站点脚本以及其分类，并说明了如何查找和修复跨站脚本漏洞的方法。
---

# 一、跨站脚本攻击（XSS）

## 1.1 什么是跨站脚本（XSS）

跨站脚本攻击XSS(Cross Site Scripting)，为了不和层叠样式表(Cascading Style Sheets, CSS)的缩写混淆，故将跨站脚本攻击缩写为XSS。攻击者可通过此漏洞来破坏用户和存在漏洞的Web应用间的交互，从而绕过网站的同源策略（同源策略可以将不同的网站隔离开来，避免相互影响），伪装成用户来执行相应的操作，并窃取用户的相关数据。如果被攻击的用户在某些应用中具有超级管理员权限，攻击者则可以完全控制应用的所有功能以及对应的数据。

## 1.2 XSS是如何攻击的

XSS的原理是通过注入恶意代码到存在漏洞的网站中，使其向用户返回恶意的JavaScript，从而恶意的JavaScript可以在用户的浏览器中执行，达到攻击者的恶意目的。

![XSS攻击原理](.gitbook/assets/cross-site-scripting.svg)

## 1.3 XSS PoC



一般情况下，我们可以通过注入payload到Web应用中，从而使得浏览器在访问Web应用时触发JavaScript运行来证明大多数的XSS漏洞。为了达到验证漏洞的目的，我们一般使用alert()函数来测试。alert函数比较简单明了、实用而无害。当成功触发alert函数时，XSS漏洞会显眼的展现在你的面前。实际上，你可以通过调用alert()来模拟受害者遭受攻击来通过大多数的XSS靶场。

不幸的是，如果你使用Chrome的话，会遇到一点小麻烦。从92版本起（2021.7.21），不同来源的iframe无法触发调用alert()。因此，你有时需要准备一些备用的PoC payload，来构造更加复杂的XSS攻击。在这种情况下，PortSwigger推荐使用print()函数来进行漏洞验证。如果你有兴趣了解更多情况，可以查看PortSwigger的[博客文章](https://portswigger.net/research/alert-is-dead-long-live-print)。

为了更好的模拟受害者使用的Chrome，PortSwigger修改了受到影响的相关靶场，以便使用Print()来证明漏洞存在。在相关的靶场提示中，PortSwigger也有对应的说明。

补充：虽然PortSwigger和大多数的教程都介绍了使用alert函数来证明漏洞，但实际情况下，在生产环境中尽量不要使用alert()函数来测试。部分的Web应用没有想象中的那么健壮，可能会因为alert()而导致对应页面无法正常渲染，影响正常用户使用（print()可能也会有相应问题）。在此，我推荐的是使用console.log()来证明漏洞存在，只需要F12打开控制台，就可以清楚的看到是否触发了XSS漏洞。

## 1.4 XSS 攻击类型

XSS 攻击类型主要有以下3种分类：

* 反射型 XSS，恶意脚本源于当前 HTTP 请求
* 存储型 XSS，恶意脚本源于网站的数据库（或其它存储数据的地方）
* DOM 型 XSS，漏洞存在于客户端而不是服务端

接下来3个小节将展开说明这3种XSS攻击。

## 1.5 反射型 XSS 漏洞

反射型 XSS 漏洞是最简单的 XSS 漏洞，它源于应用程序在接收 HTTP 请求中的数据时，以不安全的方式在响应中引用得到的数据。

下面是一个简单的反射型 XSS 漏洞案例：

`https://insecure-website.com/status?message=All+is+well.`

`<p>Status: All is well.</p>`

应用程序没有对数据数据进行任何处理，因此在这里可以尝试构建一个恶意攻击的URL：

`https://insecure-website.com/status?message=script>/*+Bad+stuff+here...+*/</script>`

`<p>Status: <script>/* Bad stuff here... */</script></p>`

如果用户打开这个恶意URL，攻击者构建的恶意脚本将在该用户的浏览器中执行，利用用户的session与应用程序交互。此时，这个脚本能执行任何操作，并获取用户可访问的任何数据。

## 1.6 存储型 XSS 漏洞

存储型 XSS 漏洞是因为应用程序接收恶意数据，存储在数据库（或其它存储数据的地方）中，并在其后的 HTTP 响应中引用该数据。

这些有问题的数据可能是通过 HTTP 请求提交给应用程序，例如在博客文章中提交的评论、聊天室中用户的昵称或客户订单中的联系方式。也有可能是从其它不受信任的来源接收的数据，比如电子邮箱应用显示通过SMTP收到的消息或邮件、营销软件显示社会媒体的文章或网络监控软件监控的网络流量数据包。

这里有一个简单的存储型 XSS 漏洞案例，留言板应用中允许用户提交留言，并显示其它用户的留言：

`<p>Hello, this is my message!</p>`

应用程序不会对这些数据进行任何处理，因此攻击者可以轻松地发送消息来攻击其它用户：

`<p><script>/* Bad stuff here... */</script></p>`

## 1.7 DOM 型XSS 漏洞

DOM 型 XSS 漏洞产生于应用程序在客户端以不安全的方式处理来自不受信任源的数据，并且会将数据写入 DOM 里。

在下面的案例中，应用程序使用 JavaScript 来从外部读取输入的字段并作为元素写到 HTML 中：

`var search = document.getElementById('search').value;`&#x20;

`var results = document.getElementById('results');`&#x20;

`results.innerHTML = 'You searched for: ' + search;`

如果攻击者可以操控这些输入字段，他们就可以轻而易举地插入恶意攻击语句，来执行自己的脚本：

You searched for: \<img src=1 onerror='/\* Bad stuff here... \*/'>

在经典案例中，输入字段通常会将 HTTP 请求部分获取，比如从 URL 查询字段获取参数，从而允许攻击者构建恶意 URL 传递攻击字符，达到类似反射型 XSS 的效果。
