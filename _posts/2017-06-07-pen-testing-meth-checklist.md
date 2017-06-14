---
layout: post
title: "安全测试Checklist"
description: "保证安全测试的系统性，避免漏测与测试结果的随机性"
tags: [web, security, test]
---
* 目录
{:toc}

## 0. 前戏 - 为什么需要安全测试Checklist

从战略层面上来说，互联网公司如何规划自己的安全蓝图呢？我们可以把安全蓝图分为三个部分：
* `Find and Fix`;
* Defend and Defer;
* Secure at the Source

其中的第一个部分就是需要我们发现并解决当前产品中的安全问题。在“Find and Fix”的过程中，可以通过漏洞扫描、渗透测试、代码审计等方式，发现系统中已知的安全问题；然后再通过设计安全方案，实施安全方案，最终解决这些问题。

本文主要是针对渗透测试开出的Checklist。在给产品做渗透测试的过程中，我们发现一个十分苦恼的问题 -- 有着不同的知识背景以及不同经验的安全测试人员发现的安全问题往往大相径庭。换句话来说，同样的产品，不同的人来做测试，测试结果往往相差很大。甚至是同一个人，给相同的产品做两轮渗透测试，测试的结果可能也有很大的不同。这其中就反应出测试的系统性不一致的问题，而此时我们就需要一个安全测试Checklist，基于这个系统的Checklist来帮我们规划安全测试，避免漏测，避免测试中发现问题的随机性，以达到规范化产品的渗透测试过程。

这不仅仅是一个Checklist，它不但列出了安全的清单，同时给出了如何做出安全的产品的方向。

## 1. 收集信息

|  测试类别 | 测试名称 | 
| :------: | :------: |
| OTG-INFO-001 | 搜索引擎信息发现和侦察 |
| OTG-INFO-002 | Fingerprint Web Server |
| OTG-INFO-003 | Review Webserver Metafiles for Information Leakage |
| OTG-INFO-004 | Enumerate Applications on Webserver |
| OTG-INFO-005 | Review Webpage Comments and Metadata for Information Leakage |
| OTG-INFO-006 | Identify application entry points |
| OTG-INFO-007 | Map execution paths through application |
| OTG-INFO-008 | Fingerprint Web Application Framework |
| OTG-INFO-009 | Fingerprint Web Application |
| OTG-INFO-010 | Map Application Architecture |


关于信息收集的部分，主要是了解测试目标，了解应用程序的实际功能与运行机制。

## 2. 配置与部署管理测试

|  测试类别 | 测试名称 | 
| :------: | :------: |
| OTG-CONFIG-001 | Test Network/Infrastructure Configuration |
| OTG-CONFIG-002 | Test Application Platform Configuration |
| OTG-CONFIG-003 | Test File Extensions Handling for Sensitive Information |
| OTG-CONFIG-004 | Backup and Unreferenced Files for Sensitive Information |
| OTG-CONFIG-005 | Enumerate Infrastructure and Application Admin Interfaces |
| OTG-CONFIG-006 | Test HTTP Methods |
| OTG-CONFIG-007 | Test HTTP Strict Transport Security |
| OTG-CONFIG-008 | Test RIA cross domain policy |


应用程序各式各样，但是一些关键的平台配置错误可能导致用同样的方法攻破不同的应用程序。

## 3. 身份管理测试

|  测试类别 | 测试名称 | 
| :------: | :------: |
| OTG-IDENT-001 | Test Role Definitions |
| OTG-IDENT-002 | Test User Registration Process |
| OTG-IDENT-003 | Test Account Provisioning Process |
| OTG-IDENT-004 | Testing for Account Enumeration and Guessable User Account |
| OTG-IDENT-005 | Testing for Weak or unenforced username policy |
| OTG-IDENT-006 | Test Permissions of Guest/Training Accounts |
| OTG-IDENT-007 | Test Account Suspension/Resumption Process |


渗透测试都是围绕用户身份进行的。

## 4. 认证测试

|  测试类别 | 测试名称 | 
| :------: | :------: |
| OTG-AUTHN-001 | Testing for Credentials Transported over an Encrypted Channel |
| OTG-AUTHN-002 | Testing for default credentials |
| OTG-AUTHN-003 | Testing for Weak lock out mechanism |
| OTG-AUTHN-004 | Testing for bypassing authentication schema |
| OTG-AUTHN-005 | Test remember password functionality |
| OTG-AUTHN-006 | Testing for Browser cache weakness |
| OTG-AUTHN-007 | Testing for Weak password policy |
| OTG-AUTHN-008 | Testing for Weak security question/answer |
| OTG-AUTHN-009 | Testing for weak password change or reset functionalities |
| OTG-AUTHN-010 | Testing for Weaker authentication in alternative channel |


你是谁的机制测试。

## 5. 授权测试

|  测试类别 | 测试名称 | 
| :------: | :------: |
| OTG-AUTHZ-001 | Testing Directory traversal/file include |
| OTG-AUTHZ-002 | Testing for bypassing authorization schema |
| OTG-AUTHZ-003 | Testing for Privilege Escalation |
| OTG-AUTHZ-004 | Testing for Insecure Direct Object References |


你能干什么的机制测试。

## 6. Session管理测试

|  测试类别 | 测试名称 | 
| :------: | :------: |
| OTG-SESS-001 | Testing for Bypassing Session Management Schema |
| OTG-SESS-002 | Testing for Cookies attributes |
| OTG-SESS-003 | Testing for Session Fixation |
| OTG-SESS-004 | Testing for Exposed Session Variables |
| OTG-SESS-005 | Testing for Cross Site Request Forgery |
| OTG-SESS-006 | Testing for logout functionality |
| OTG-SESS-007 | Test Session Timeout |
| OTG-SESS-008 | Testing for Session puzzling |


在无状态下维持你是谁你能干什么机制的测试。

## 7. 数据验证测试

|  测试类别 | 测试名称 | 
| :------: | :------: |
| OTG-INPVAL-001 | Testing for Reflected Cross Site Scripting |
| OTG-INPVAL-002 | Testing for Stored Cross Site Scripting |
| OTG-INPVAL-003 | Testing for HTTP Verb Tampering |
| OTG-INPVAL-004 | Testing for HTTP Parameter pollution |
| OTG-INPVAL-005 | Testing for SQL Injection |
|  | Oracle Testing |
|  | MySQL Testing |
|  | SQL Server Testing |
|  | Testing PostgreSQL |
|  | MS Access Testing |
|  | Testing for NoSQL injection |
| OTG-INPVAL-006 | Testing for LDAP Injection |
| OTG-INPVAL-007 | Testing for ORM Injection |
| OTG-INPVAL-008 | Testing for XML Injection |
| OTG-INPVAL-009 | Testing for SSI Injection |
| OTG-INPVAL-010 | Testing for XPath Injection |
| OTG-INPVAL-011 | IMAP/SMTP Injection |
| OTG-INPVAL-012 | Testing for Code Injection |
|  | Testing for Local File Inclusion |
|  | Testing for Remote File Inclusion |
| OTG-INPVAL-013 | Testing for Command Injection |
| OTG-INPVAL-014 | Testing for Buffer overflow |
|  | Testing for Heap overflow |
|  | Testing for Stack overflow |
|  | Testing for Format string |
| OTG-INPVAL-015 | Testing for incubated vulnerabilities |
| OTG-INPVAL-016 | Testing for HTTP Splitting/Smuggling |


一切不信任问题的来源。

## 8. 错误处理

|  测试类别 | 测试名称 | 
| :------: | :------: |
| OTG-ERR-001 | Analysis of Error Codes |
| OTG-ERR-002 | Analysis of Stack Traces |


出错后常会暴露系统内部信息。

## 9. 密码学
|  测试类别 | 测试名称 | 
| :------: | :------: |
| OTG-CRYPST-001 | Testing for Weak SSL/TSL Ciphers, Insufficient Transport Layer Protection |
| OTG-CRYPST-002 | Testing for Padding Oracle |
| OTG-CRYPST-003 | Testing for Sensitive information sent via unencrypted channels |


保证机密性的核心机制。

## 10. 业务逻辑测试

|  测试类别 | 测试名称 | 
| :------: | :------: |
| OTG-BUSLOGIC-001 | Test Business Logic Data Validation |
| OTG-BUSLOGIC-002 | Test Ability to Forge Requests |
| OTG-BUSLOGIC-003 | Test Integrity Checks |
| OTG-BUSLOGIC-004 | Test for Process Timing |
| OTG-BUSLOGIC-005 | Test Number of Times a Function Can be Used Limits |
| OTG-BUSLOGIC-006 | Testing for the Circumvention of Work Flows |
| OTG-BUSLOGIC-007 | Test Defenses Against Application Mis-use |
| OTG-BUSLOGIC-008 | Test Upload of Unexpected File Types |
| OTG-BUSLOGIC-009 | Test Upload of Malicious Files |


跳出固定思维模式对应用进行测试。

## 11. 客户端测试

|  测试类别 | 测试名称 | 
| :------: | :------: |
| OTG-INPVAL-001 | Testing for DOM based Cross Site Scripting |
| OTG-INPVAL-002 | Testing for JavaScript Execution |
| OTG-INPVAL-003 | Testing for HTML Injection |
| OTG-INPVAL-004 | Testing for Client Side URL Redirect |
| OTG-INPVAL-005 | Testing for CSS Injection |
| OTG-INPVAL-006 | Testing for Client Side Resource Manipulation |
| OTG-INPVAL-007 | Test Cross Origin Resource Sharing |
| OTG-INPVAL-008 | Testing for Cross Site Flashing |
| OTG-INPVAL-009 | Testing for Clickjacking |
| OTG-INPVAL-010 | Testing WebSockets |
| OTG-INPVAL-011 | Test Web Messaging |
| OTG-INPVAL-012 | Test Local Storage |

测试客户端如何执行代码，一般来说，客户端是浏览器或者浏览器插件。

## 12. 实践
{% include image.html path="documentation/pen-test/Authentication-Checklist-Demo.png" path-detail="documentation/pen-test/Authentication-Checklist-Demo.png" alt="Authentication-Checklist-Demo" %}


Checklist内容来自[OWASP Testing Checklist](https://www.owasp.org/index.php/Testing_Checklist),详细请参考[OWASP 测试指南](https://kennel209.gitbooks.io/owasp-testing-guide-v4/zh/index)


