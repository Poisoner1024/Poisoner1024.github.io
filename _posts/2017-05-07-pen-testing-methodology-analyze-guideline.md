---
layout: post
title: "Web应用程序渗透测试方法论 - 分析应用程序"
description: "分析应用程序，为进一步的测试方向做指引"
tags: [web, security, test]
---
注：原文来自《黑客攻防技术宝典》

* 目录
{:toc}

{% include image.html path="documentation/pen-test/analyze-app.png" path-detail="documentation/pen-test/analyze-app.png" alt="analyze-app" %}

## 1. 确定功能
(1) 确定为使应用程序正常运行而建立的核心功能以及每项功能旨在执行的操作。

(2) 确定应用程序采用的核心安全机制以及它们的工作机制。重点了解处理身份验证、会话管理与访问控制的关键机制以及支持它们的功能，如用户注册和账户恢复。

(3) 确定所有较为外围的功能和行为，如重定向使用、站外链接、错误消息、管理与日志功能。

(4) 确定任何与应用程序在其他地方使用的标准GUI外观、参数命名或导航机制不一致的功能，然后将其挑选出来以进行深入测试。

## 2. 确定数据进入点
(1) 确定在应用程序中引入用户输入的所有进入点，包括URL、查询字符串参数、POST数据、Cookie与其他由应用程序处理的HTTP消息头。

(2) 分析应用程序使用的所有定制数据传输或编码机制，如非常规的查询字符串格式。了解被提交的数据是否包含参数名与参数值，或者是否使用了其他表示方法。

(3) 确定所有在应用程序中引入用户可控制或其他第三方数据的带外通道，例如，处理和显示通过SMTP收到的消息的Web邮件应用程序。

## 3. 确定所使用的技术
(1) 确定客户端使用的各种不同技术，如表单、脚本、Cookie、Java Applet、ActiveX控件与Flash对象。

(2) 尽可能确定服务器端使用的技术，包括脚本语言、应用程序平台以及与数据库和电子邮件系统等后端组件的交互。

(3) 检查在应用程序响应中返回的HTTP Server消息头，查找定制HTTP消息头或HTML源代码注释中出现的其他任何软件标识符。注意，有时候，不同的应用程序区域由不同的后端组件处理，因此渗透测试员可能会收到不同的标识符。

(4) 运行Httprint工具，为Web服务器做“指纹标识”。

(5) 检查内容解析过程中获得的结果，确定所有有助于了解服务器端使用技术的文件扩展名、目录或其他URL序列。检查所有会话令牌和其他Cookie的名称。同时，使用Google搜索与这些内容有关的技术。

(6) 确定任何看似有用的、属于第三方代码组件的脚本名称与查询字符串参数。在Google中使用inurl:限定词搜索这些内容，查找所有使用相同脚本与参数、并因此使用相同第三方组件的应用程序。对这些站点进行非入侵式的审查，这样做可能会发现其他在攻击的应用程序中没有明确链接的内容和功能。

## 4. 解析受攻击面
(1) `设法确定服务器端应用程序的内部结构与功能以及用于实现客户端可见行为的后台机制`。例如，获取客户订单的功能可能与数据库进行交互。

(2) `确定各种与每一项功能有关的常见漏洞`。例如，文件上传功能可能易于受到路径遍历攻击；用户间通信可能易于受到XSS攻击；“联系我们”功能可能易于受到SMTP注入攻击。

(3) `制定一个攻击计划，优先考虑最有用的功能以及与它有关的最严重的潜在漏洞`。使用这份计划作为指导，决定在不同的区域投入多少时间和精力。



















