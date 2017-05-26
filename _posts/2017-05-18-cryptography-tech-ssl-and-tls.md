---
layout: post
title: "密码技术 - SSL/TLS"
description: "为了更安全的通信"
tags: [security, cryptography]
---

* 目录
{:toc}

## 1. 什么是SSL/TLS

### 1.1 Alice在Bob书店买书
Bob书店是Alice经常关顾的一家网店，因为在Bob书店她可以搜索到新出版的图书，还可以通过信用卡快速完成支付，购买的书还能快递到家，真的很方便。

有一天，Alice读了一本关于网络信息安全的书，书上说“互联网上传输的数据都是可以被窃听的”。Alice感到非常担心，自己购买新书的时候输入的**信用卡号会不会被窃听呢？**

Alice看到Bob的书店写着“所有的数据都是通过SSL/TLS发送”，所有的请求都是以https://开头的，这是一种安全的方式，但是Alice还是不放心，到底在我的Web浏览器和Bob书店的网址之间都发生了哪些事呢？

### 1.2 客户端与服务器端
Alice和Bob书店的通信过程示意图：
<div align='center'>
{% include image.html path="documentation/cryptography/client-server-communication.png" path-detail="documentation/cryptography/client-server-communication.png" alt="client-server-communication" %}
</div>

不使用SSL/TLS发送信用卡号的情形：
<div align='center'>
{% include image.html path="documentation/cryptography/communication-without-ssl.png" path-detail="documentation/cryptography/communication-without-ssl.png" alt="communication-without-ssl" %}
</div>

### 1.3 用SSL/TLS承载HTTP
用SSL/TLS承载HTTP，对请求和响应进行加密：
<div align='center'>
{% include image.html path="documentation/cryptography/communicate-with-ssl.png" path-detail="documentation/cryptography/communicate-with-ssl.png" alt="communicate-with-ssl" %}
</div>

### 1.4 SSL/TLS的工作
在大致了解了SSL/TLS之后，我们来整理一下SSL/TLS到底负责哪些工作。我们想要实现的是通过本地的Web浏览器访问网络上的Web服务器，并进行安全的通信。拿Alice的例子来说，Alice希望通过Web浏览器向Bob书店发送信用卡号。在这里，我们有几个必须要解决的问题。
* (1) Alice的信用卡号和地址在发送到Bob书店的过程中不能被窃听。
* (2) Alice的信用卡号和地址在发送到Bob书店的过程中不能被篡改。
* (3) 确认通信对方的Web服务器是真正的Bob书店。

在这里，(1)是`机密性`问题，(2)是`完整性`问题，而(3)则是`认证`的问题。

为了解决这些问题，我们在密码学家的工具箱中找一找。

要确保机密性，可以使用**对称密码**。由于对称密码的密钥不能被攻击者预测，因此我们使用**伪随机数生成器**来生成密钥。若要将对称密码的密钥发送给通信对象，可以使用**公钥密码**或者**Diffie-Hellman密码交换**。

要识别篡改，对数据进行认证，可以使用**消息认证码**。消息认证码是使用**单项散列函数**来实现的。

要对通信对象进行认证，可以使用对公钥加上**数字签名**所生成的证书。

好，工具已经找齐了，下面只要用一个“框架”（framework）将这些工具组合起来就可以了。`SSL/TLS协议其实就扮演了这样一种框架的角色。`
### 1.5 SSL/TLS也可以保护其他协议
SSL/TLS上面不仅可以承载HTTP，还可以承载其他很多协议。
<div align='center'>
{% include image.html path="documentation/cryptography/ssl-with-diff-protocols.png" path-detail="documentation/cryptography/ssl-with-diff-protocols.png" alt="ssl-with-diff-protocols" %}
</div>

### 1.6 密码套件
SSL/TLS提供了一种密码通信的框架，这意味着SSL/TLS中使用的对称密码、公钥密码、数字签名、单向散列函数等技术，都是可以像零件一样进行替换的。也就是说，如果发现现在所使用的某个密码技术存在弱点，那么只要将这一部分进行替换就可以了。

尽管如此，也并不是说所有的组件都可以自由选择。由于实际进行对话的客户端和服务端必须使用相同的密码技术才能进行通信，因此如果选择过于自由，就难以确保整体的兼容性。为此，SSL/TLS就像事先搭配好的盒饭一样，规定了一些密码技术的“推荐套餐”，这种推荐套餐称为**密码套件**（cipher suite)。

### 1.7 SSL与TLS的区别
[SSL与TLS的发展历史介绍](https://zh.wikipedia.org/wiki/%E5%82%B3%E8%BC%B8%E5%B1%A4%E5%AE%89%E5%85%A8%E5%8D%94%E8%AD%B0)，很多时候我们将SSL和TLS作为一个整体(SSL/TLS)来对待。

## 2. 使用SSL/TLS进行通信
### 2.1 层次化的协议
**TLS协议**是由`TLS记录协议`和`TLS握手协议`这两层协议叠加而成的。位于底层的TLS记录协议负责进行加密，而位于上层的TLS握手协议负责除了加密以外的其他各种操作。上层的TLS握手协议又可以分为4个子协议。TLS协议的层次结构如下：
<div align='center'>
{% include image.html path="documentation/cryptography/TLS-Layers-Structure.png" path-detail="documentation/cryptography/TLS-Layers-Structure.png" alt="TLS-Layers-Structure" %}
</div>

* 1 TLS 记录协议，位于TLS握手协议的下层，是负责使用对称密码对消息进行加密通信的部分。TLS记录协议中使用了对称密码和消息认证码，但是具体的算法和共享密钥则是通过后面将要介绍的握手协议在服务器和客户端之间协商决定的。
* 2 TLS握手协议
    - 2-1 握手协议，是TLS握手协议的一部分，负责在客户端和服务器之间协商决定密码算法和共享密钥。基于证书的认证操作也是在这个协议中完成。它是4个子协议中最负责的一个。
    - 2-2 密码规格变更协议，是TLS握手协议的一部分，负责向通信对象传达变更密码方式的信号。简单地说，就跟向对方喊“1，2，3！”差不多。
    - 2-3 警告协议，是TLS握手协议的一部分，负责在发生错误时将错误传达给对方。
    - 2-4 应用数据协议，是TLS握手协议的一部分，负责将TLS上面承载的应用数据传达给通信对象。

### 2.2 1 TLS 记录协议
**TLS记录协议**负责消息的压缩、加密以及数据的认证，其处理过程如下：
<div align='center'>
{% include image.html path="documentation/cryptography/tls-record-protocol-process.png" path-detail="documentation/cryptography/tls-record-protocol-process.png" alt="tls-record-protocol-process" %}
</div>
**首先，**消息被分割成多个较短的片段,然后分别对每个片段进行压缩。压缩算法需要与通信对象协商决定。

**接下来**，经过压缩的片段会被加上消息认证码，这是为了保证完整性，并进行数据的认证。通过附加消息认证码的MAC值，可以识别出篡改。与此同时，为了防止重放攻击，在计算消息认证码时，还加上了片段的编码。单向散列函数的算法，以及消息认证码所使用的共享密钥都需要与通信对象协商决定。

**再接下来**，经过压缩的片段再加上消息认证码会一起通过对称密码进行加密。加密使用CBC模式，CBC模式的初始化向量（IV）通过主密码（master secret）生成，而对称密码的算法以及共享密码需要与通信对象协商决定。

**最后**，上述经过加密的数据再加上由数据类型、版本号、压缩后的长度组成的报头就是最终的报文数据。其中，数据类型为TLS记录协议所承载的4个子协议的其中之一。

### 2.3 2-1 握手协议
**握手协议**，是TLS握手协议的一部分，负责生成共享密钥以及交互证书。其中，生成共享密钥是为了进行密码通信，交互证书是为了通信双方相互进行认证。

握手协议这一名称中的“握手”，是服务器和客户端在密码通信之前交换一些必要信息这一过程的比喻。

由于握手协议中的信息交换是在没有加密的情况下进行的（即使用“不加密”这一密码套件），也就是说，在这一协议中所收发的所有数据都可能被窃听者Eva窃听，因此在这一过程中必须使用公钥密码或者Diffie-Hellman密钥交换。

握手协议的过程如图：
{% include image.html path="documentation/cryptography/tls-handshake-protocol-process.png" path-detail="documentation/cryptography/tls-handshake-protocol-process.png" alt="tls-handshake-protocol-process" %}

`从结果来看，握手协议完成了下列操作：`
{% highlight markdown %}
* 客户端获得了服务器的合法公钥，完成了服务器认证
* 服务器获得了客户端的合法公钥，完成了客户端认证(当需要客户端认证时)
* 客户端和服务器生成了密码通信中使用的共享密钥
* 客户端和服务器生成了消息认证码中使用的共享密钥
{% endhighlight %}

### 2.4 2-2 密码规格变更协议
密码规格变更协议(change cipher spec protocol)用于密码切换的同步。

那么为什么这个协议不叫密码规格*开始*协议，而是要叫做密码规格*变更*协议呢？这是因为即便在密码通信开始之后，客户端和服务器也可以通过重新握手来再次改变密码套件。也就是说，在最开始的时候，客户端和服务器是使用“不加密”这一密码套件进行通信的，因此通信内容没有进行加密。

### 2.5 2-3 警告协议
警告协议(alert protocol)用于当发生错误时通知通信对象。当握手协议的过程中产生异常，或者发生消息认证码错误、压缩数据无法解压缩等问题时，会使用该协议。

### 2.6 2-4 应用数据协议
应用数据协议用于和通信对象之间传送应用数据。

当TLS承载HTTP时，HTTP的请求和响应就会通过TLS的应用数据协议和TLS记录协议来进行传送。

### 2.7 主密码
主密码是TLS客户端和服务器之间协商出来的一个秘密的数值。这个数值非常重要，TLS密码通信的机密性和数据的认证全部依靠这个数值。主密码是一个48字节（384比特）的数值。

`主密码是如何计算出来的`？主密码是客户端和服务器根据下列信息计算出来的：
* 预备主密码
* 客户端随机数
* 服务器端随机数

当使用RSA公钥密码时，客户端会在发送ClientKeyExchange消息时，将经过加密的预备主密码一起发送给服务器。

当使用Diffie-Hellman密钥交换时，客户端会在发送ClientKeyExchange消息时，将Diffie-Hellman的公开值一起发送给服务器。根据这个值，客户端和服务器端会各自生成预备主密码。关于客户端和服务器为什么能够生成出相同的值，请参照Diffie-Hellman密钥交换算法。

客户端随机数和服务器端随机数的作用相当于防止攻击者事先计算出密钥的盐。

当根据预备主密码计算主密码时，需要使用基于密钥套件中定义的单项散列函数（如SHA-256）来实现的伪随机函数（Pseudo Random Function）。

密钥素材的依赖关系如下：
{% include image.html path="documentation/cryptography/key-material-depend-relationship.png" path-detail="documentation/cryptography/key-material-depend-relationship.png" alt="key-material-depend-relationship" %}

`主密码的目的是什么？`主密码用于生成下列6种信息。
* 对称密码的密钥（客户端 -> 服务器）
* 对称密码的密钥（客户端 <- 服务器）
* 消息认证码的密钥（客户端 -> 服务器）
* 消息认证码的密钥（客户端 <- 服务器）
* 对称密码的CBC模式所使用的初始化向量（客户端 -> 服务器）
* 对称密码的CBC模式所使用的初始化向量（客户端 <- 服务器）

### 2.8 TLS中使用的密码技术小结
TLS握手协议中使用的密码技术：

| 密码技术 | 作用 |
| :------: | :------ |
| 公钥密码 | 加密预备主密码 |
| 单项散列函数 | 构成伪随机数生成器 |
| 伪随机数生成器 | 生成预备主密码；根据主密码生成密钥（密码参数）；生成初始化向量 |

<br>
TLS记录协议中使用的密码技术：

| 密码技术 | 作用 |
| :------: | :------ |
| 对称密码（CBC模式） | 确保片段的机密性 |
| 消息认证码 | 确保片段的完整性并进行认证 |
| 认证加密（AEAD） | 确保片段的完整性和机密性并进行认证 |

<br>
## 3. 对SSL/TLS的攻击

### 3.1 对各个密码技术的攻击

### 3.2 OpenSSL的心脏出血漏洞

### 3.3 SSL 3.0的漏洞与POODLE攻击

### 3.4 FREAK攻击与密码产品出口管制

### 3.5 对伪随机数生成器的攻击

### 3.6 利用证书的时间差进行攻击

## 4. SSL/TLS用户的注意事项

### 4.1 不要误解证书的含义

### 4.2 密码通信之前的数据是不受保护的

### 4.3 密码通信之后的数据是不受保护的



















