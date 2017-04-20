---
layout: post
title: "客户端脚本安全"
description: "常见的客户端脚本安全问题描述"
tags: [web, security]
---
注：原文来自《白帽子讲Web安全》

* 目录
{:toc}

## 1. 浏览器安全

### 同源策略
`同源策略`（Same Origin Policy）是一种约定，它是浏览器最核心也是最基本的安全功能，如果缺少了同源策略，则浏览器的正常功能可能都会受到影响。可以说Web是构建在同源策略基础之上的，浏览器只是针对同源策略的一种实现。

**浏览器的同源策略，限制了来自不同源“document”或脚本，对当前“document”读取或设置某些属性**

如果没有同源策略，可能a.com 的一段JavaScript脚本，在b.com 未曾加载此脚本时，也可以随意涂改b.com 的页面（在浏览器的显示中）。为了不让浏览器的页面行为发生混乱，浏览器提出了“Origin”（源）这一概念，来自不同Origin 的对象无法相互干扰。
<div align='center'>
{% include image.html path="documentation/white-hat-web-security/javascript-same-origin-policy.png" path-detail="documentation/white-hat-web-security/javascript-same-origin-policy.png" alt="javascript-same-origin-policy" %}
</div>
🈶上表可以看出，影响“源”的因素有：`host`（域名或IP地址，如果是IP地址则看做一个根域名）、`子域名`、`端口`、`协议`。

**需要注意的是，**对于当前页面来说，页面内存放JavaScript文件的域并不重要，重要的是加载JavaScript页面所在的域是什么。换言之，a.com通过以下代码：
{% highlight javascript %}
<script src='http://b.com/b.js'></script>
{% endhighlight %}
加载了b.com 上的b.js，但是b.js 是运行在a.com 页面中的，因此对于当前打开的页面（a.com页面）来说，b.js 的Origin 就应该是 a.com 而非b.com。

在浏览器中，script/img/iframe/link 等标签都可以跨域加载资源，而不受同源策略的限制。这些带“src”属性的标签每次加载时，实际上是由浏览器发起了一次GET请求。不同意XMLHttpRequest 的是，通过src 属性加载的资源，浏览器限制了JavaScript的权限，使其不能读、写返回的内容。

而对于XMLHttpRequest来说，它可以访问来自同源对象的内容，但XMLHttpRequest受到同源策略的约束，不能跨域访问资源，在AJAX应用的开发中尤其需要注意这一点。如果XMLHttpRequest能够跨域访问资源，则可能会导致一些敏感数据泄露，比如CSRF 的token，从而导致发生安全问题。

但是互联网是开放的，随着业务的发展，跨域请求的需求越来越迫切，因此W3C委员会制定了XMLHttpRequest跨域访问标准。它需要通过目标域返回的HTTP头来授权是否允许跨域访问，因为HTTP头对于JavaScript来说一般是无法控制的，所以认为这个方案可以实施。`注意`：这个跨域访问方案的安全基础就是信任“JavaScript 无法控制该HTTP头”，如果此信任基础被打破，则此方案也将不再安全。

### 浏览器沙箱
针对客户端的攻击今年来呈爆发趋势，在网页中插入一段恶意代码，利用浏览器漏洞执行任意代码的攻击方式，在黑客圈子被形象地称为“挂马”。

“挂马”是浏览器需要面对的一个主要威胁。近年来，独立于杀毒软件之外，浏览器厂商根据挂马的特点研究出了一些对抗挂马的技术。比如在Windows 系统中，浏览器密切结合DEP、ASLR、SafeSEH 等操作系统提供的保护技术，对抗内存攻击。与此同时，浏览器还发展出了多进程架构，从安全性上很了很大的提供。

浏览器的多进程架构，将浏览器的各个功能模块分开，各个浏览器实例分开，当一个进程崩溃时，也不会影响到其他的进程。

Google Chrome 是第一个采取多进程架构的浏览器。Google Chrome 的主要进程分为：浏览器进程、渲染进程、插件进程、扩展进程。插件进程如 flash、java、pdf等与浏览器进程严格隔离，因此不会相互影响。
<div align='center'>
{% include image.html path="documentation/white-hat-web-security/chrome-security-arch.jpg" path-detail="documentation/white-hat-web-security/chrome-security-arch.jpg" alt="chrome-security-arch" %}
</div>
渲染引擎由Sandbox隔离，网页代码要与浏览器内核进程通信、与操作系统通信都需要通过IPC channel，在其中会进行一些安全检查。

Sandbox 即沙箱，计算机技术发展到今天，Sandbox 已经成为泛指“资源隔离类模块”的代名词。Sandbox 的设计目的一般是为了让不可信任的代码运行在一定的环境中，限制不可信任的代码访问隔离区之外的资源。如果一定要跨越Sandbox 便捷产生数据交换，则只能通过制定的数据通道，比如经过封装的API 来完成，在这些API 中会严格检查请求的合法性。

Sandbox 的应用范围非常广泛。比如一个提供hosting 服务的共享主机环境，假设支持用户上传PHP、Python、Java等语言的代码，为了防止用户代码破坏系统环境，或者是不同用户之间的代码相互影响，则应该设计一个Sandbox 对用户代码进行隔离。Sandbox 需要考虑用户代码针对文件系统、内存、数据库、网络的可能请求，可以采用默认拒绝的策略，对于有需要的请求，则可以通过封装API的方式实现。

对于浏览器来说，采用Sandbox 技术，无疑可以让不受信任的网页代码、JavaScript代码运行在一个受到限制的环境中，从而保护本地桌面系统的安全。Google Chrome实现了一个相对完整的Sandbox：
<div align='center'>
{% include image.html path="documentation/white-hat-web-security/chrome-sandbox.png" path-detail="documentation/white-hat-web-security/chrome-sandbox.png" alt="chrome-sandbox" %}
</div>
多进程架构最明显的一个好处是，相对于单进程浏览器，在发生崩溃时，多进程浏览器只是会崩溃当前的Tab页，而单进程浏览器则会崩溃整个浏览器进程。这对于用户体验是很大的提升。

但是浏览器安全是一个整体，在现今的流浪器中，虽然有多进程架构和Sandbox 的保护，但是浏览器所加载的一些第三方插件却往往不受Sandbox 管辖。比如近年来在Pwn2Own大会上被攻克的浏览器，往往都是由于加载的第三方插件出现安全漏洞导致的。Flash、Java、PDF、.Net Framework 在近年来都成为浏览器攻击的热点。

也许在不远的未来，在浏览器的安全模式中会更加重视这些第三方插件，不同厂商之间会就安全达成一致的标准，也只有这样，才能将这个互联网的入口打造得更加牢固。

### 恶意网站拦截
上节提到了“挂马”攻击方式能够破坏浏览器安全，在很多时候，“挂马”攻击在实施时会在一个正常的网页中通过script或者iframe等标签加载一个恶意网址。而除了挂马所加载的恶意网址之外，钓鱼网站、诈骗网站对于用户来说也是一种恶意网址。为了保护用户安全，浏览器厂商纷纷推出了各自的拦截恶意网址功能。目前各个浏览器的拦截恶意网址的功能都是基于“黑名单”的。

恶意网址拦截的`工作原理`很简单，一般都是浏览器周期性地从服务器端获取一份最新的恶意网址黑名单，如果用户上网时访问的网址存在于此黑名单中，浏览器就会弹出一个警告页面。
<div align='center'>
{% include image.html path="documentation/white-hat-web-security/Google-Chrome-warning.png" path-detail="documentation/white-hat-web-security/Google-Chrome-warning.png" alt="Google-Chrome-warning" %}
</div>

常见的恶意网址分为两类：1、挂马网站，这些网站通常包含有恶意的脚本如JavaScript 或Flash，通过利用浏览器的漏洞（包括一些插件、控件漏洞）执行shellcode，在用户电脑中植入木马；2、钓鱼网站，通过模仿知名网站的相似页面来欺骗用户。

## 2. 跨站脚本攻击（XSS）

### XSS 简介
跨站脚本攻击，英文全称是Cross Site Script，本来的缩写是CSS，但是为了和层叠样式表（Cascading Style Sheet, CSS）有所区别，所以在安全领域叫做“XSS”。

**XSS攻击，通常指黑客通过“HTML 注入”篡改了网页，插入了恶意的脚本，从而在用户浏览网页时，控制用户浏览器的一种攻击。**在一开始，这种攻击的演示案例是跨域的，所以叫做“跨站脚本”。但是发展到今天，由于JavaScript的强大功能以及网站前端应用的复杂化，是否跨域已经不再重要。但是由于历史原因，XSS 这个名字却一直保留下来。

XSS 长期以来被列为客户端Web安全中的头号大敌。因为XSS 破坏力强大，且产生的场景复杂，难以一次性解决。现在业内达成的共识是：针对各种不同场景产生的XSS，需要区分情景对待。即便如此，复杂的应用环境仍然是XSS 滋生的温床。

XSS 根据效果的不同可以分成如下几类：
* 1. 反射型XSS
  反射型XSS 只是简单地把用户输入的数据“反射”给浏览器。也就是说，黑客往往需要诱使用户“点击”一个恶意链接，才能攻击成功。

  反射型XSS 也叫做“非持久型 XSS”（Non-persistent XSS）。

* 2. 存储型XSS
  存储型XSS 会把用户输入的数据“存储”在服务器端。这种XSS 具有很强的稳定性。比较常见的一个场景就是，黑客写下一篇包含恶意JavaScript 代码的博客文章，文章发表后，所有访问该博客文章的用户，都会在他们的浏览器中执行这段恶意的JavaScript代码。黑客把恶意的脚本保存到服务器端，所以这种XSS 攻击就叫做“存储型 XSS”。

  存储型 XSS 通常也叫做“持久型 XSS”（Persistent XSS），因为从效果上来说，它存在的时间是比较长的。

* 3. DOM Based XSS
  实际上，这种类型的XSS 并非按照“数据是否保存在服务器端”来划分，DOM Based XSS 从效果上来说也是反射型XSS。单独划分出来，是因为DOM Based XSS的形成原因比较特别，发现它的安全专家专门提出了这种类型的XSS。出于历史原因，也就把它单独作为一个分类了。

  通过修改页面的DOM节点形成的XSS，称之为DOM Based XSS。

### XSS 攻击平台
* [Attack API](http://pdp.soup.io/post/7429829/Bring-Back-the-Attack-to-the-API)
  Attack API 是安全研究中pdp所主导的一个项目，它总结了很多能够直接使用XSS Payload，归纳为API的方式
  
* [BeEF](http://beefproject.com/)
  BeEF曾经是最好的XSS 演示平台。不同于Attack API，BeEF所演示的是一个完整的攻击过程。BeEF有一个控制后台，攻击者可以在后台控制前端的一切。每个被XSS 攻击的用户都将出现在后台，后台控制者就可以控制这些浏览器的行为，并可以通过XSS 向这些用户发送命令。

* [XSS-Proxy](http://xss-proxy.sourceforge.net/)
  XSS-Proxy是一个轻量级的XSS 攻击平台，通过嵌套iframe的方式可以实时地远程控制被XSS 攻击的浏览器。

这些XSS 攻击平台有助于深入理解XSS 的原理和危害。

## 3. 跨站点请求伪造（CSRF）

### CSRF 简介
CSRF 的全名是Cross Site Request Forgery。先来看一个例子。

假设现在Sohu 博客有个XSS 问题，只需要请求下面这个URL，就能够把编号为“156713012”的博客文章删除。
{% highlight URL %}
http://blog.sohu.com/manage/entry.do?m=delete&id=156713012
{% endhighlight %} 

这个URL同时还存在CSRF 漏洞。我们将尝试利用CSRF 漏洞，删除编号为“156713012”的博客文章。
* 1. 攻击者首先在自己的域构造一个页面：
{% highlight URL %}
http://www.a.com/csrf.html
{% endhighlight %} 

其内容为：
{% highlight JavaScript %}
<img src="http://blog.sohu.com/manage/entry.do?m=delete&id=156713012" />
{% endhighlight %} 

* 2. 攻击者诱使目标用户，也就是博客主访问这个页面，已执行CSRF 攻击。
在博客主访问http://www.a.com/csrf.html时，图片标签向搜狐的服务器发送了一次GET请求，而这次请求，导致了搜狐博客上的一篇文章被删除。

回顾下整个攻击过程，攻击者仅仅诱使用户访问了一个页面，就使该用户身份在第三方站点里执行了一次操作。试想：如果这张图片是展示在某个论坛、某个博客，甚至搜狐的一些用户控件中，会产生什么效果呢？只需要经过精心的设计，就能够起到更大的破坏作用。

这个删除博客文章的请求，是攻击者所伪造的，所以这种攻击就叫做“跨站点请求伪造”。

### CSRF 本质
CSRF 为什么能够攻击成功？其本质原因是`重要操作的所有参数都是可以被攻击者猜测到的`。

攻击者只有预测出URL 的所有参数与参数值，才能成功地构造一个伪造的请求；反之，攻击者将无法攻击成功。

## 4. 点击劫持（ClickJacking）
2008年，安全专家Robert Hansen 与 Jeremiah Grossman发现了一种被他们称为“ClickJacking”（点击劫持）的攻击，这种攻击方式影响了几乎所有的桌面平台，包括IE、Safari、Firefox、Opera以及Adobe Flash。

点击劫持是一种视觉上的欺骗手段。攻击者使用一个透明的、不可见的iframe，覆盖在一个网页上，然后诱使用户在该网页上进行操作，此时用户将在不知情的情况下点击透明的iframe页面。通过调整iframe页面的位置，可以诱使用户恰好点击在iframe页面的一些功能性按钮上。

点击劫持攻击与CSRF 攻击有异曲同工之妙，都是在用户不知情的情况下诱使用户完成一些动作。但是在CSRF 攻击的过程中，如果出现用户交互的页面，则攻击可能会无法顺利完成。与之相反的是，点击劫持没有这个顾虑，它利用的就是与用户产生交互的页面。

点击劫持攻击的类型有：
* Flash 点击劫持；
* 图片覆盖攻击；
* 拖拽劫持与数据窃取；
* ClickJacking 3.0:屏幕劫持；

针对传统的ClickJacking，一般可以通过禁止跨域的iframe来防范。

## 5. HTML 5安全
HTML 5带来了新的功能，也带来了新的安全挑战。

### HTML 5新标签
* 新标签的XSS
* iframe的sandbox
* Link Types: noreferrer
* Canvas的妙用

### HTML 5其他安全问题
* Cross-Origin Resource Sharing
* postMessage - 跨窗口传递消息
* Web Storage