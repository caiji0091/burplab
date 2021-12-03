---
description: 本章将介绍什么跨站点脚本以及其分类，并说明了如何查找和修复跨站脚本漏洞的方法。
---

# 一.跨站脚本攻击（XSS）

## 1.1 什么是跨站脚本（XSS）

跨站脚本攻击XSS(Cross Site Scripting)，为了不和层叠样式表(Cascading Style Sheets, CSS)的缩写混淆，故将跨站脚本攻击缩写为XSS。攻击者可通过此漏洞来破坏用户和存在漏洞的Web应用间的交互，从而绕过网站的同源策略（同源策略可以将不同的网站隔离开来，避免相互影响），伪装成用户来执行相应的操作，并窃取用户的相关数据。如果被攻击的用户在某些应用中具有超级管理员权限，攻击者则可以完全控制应用的所有功能以及对应的数据。

## 1.2 XSS是如何攻击的

XSS的原理是通过注入恶意代码到存在漏洞的网站中，使其向用户返回恶意的JavaScript，从而恶意的JavaScript可以在用户的浏览器中执行，达到攻击者的恶意目的。

![XSS攻击原理](.gitbook/assets/cross-site-scripting.svg)

## 1.3 XSS PoC



一般情况下，我们可以通过注入payload到Web应用中，从而使得浏览器在访问Web应用时触发JavaScript运行来证明大多数的XSS漏洞。为了达到验证漏洞的目的，我们一般使用alert()函数来测试。alert函数比较简单明了、实用而无害。当成功触发alert函数时，XSS漏洞会显眼的展现在你的面前。实际上，你可以通过调用alert()来模拟受害者遭受攻击来通过大多数的XSS靶场。

不幸的是，如果你使用Chrome的话，会遇到一点小麻烦。从92版本起（2021.7.21），不同来源的iframe无法触发调用alert()。因此，你有时需要准备一些备用的PoC payload，来构造更加复杂的XSS攻击。在这种情况下，PortSwigger推荐使用print()函数来进行漏洞验证。如果你有兴趣了解更多情况，可以查看PortSwigger的[博客文章](https://portswigger.net/research/alert-is-dead-long-live-print)。

为了更好的模拟受害者使用的Chrome，PortSwigger修改了受到影响的相关靶场，以便使用Print()来证明漏洞存在。在相关的靶场提示中，PortSwigger也有对应的说明。

补充：虽然PortSwigger和大多数的教程都介绍了使用alert函数来证明漏洞，但实际情况下，在生产环境中尽量不要使用alert函数来测试。部分的Web应用没有想象中的那么健壮，可能会因为alert而导致对应页面无法正常渲染，影响正常用户使用（print可能也会有相应问题）。在此，我推荐的是使用console.log()来证明漏洞存在，只需要F12打开控制台，就可以清楚的看到是否触发了XSS漏洞。
