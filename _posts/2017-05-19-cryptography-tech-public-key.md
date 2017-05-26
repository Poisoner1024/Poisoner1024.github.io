---
layout: post
title: "密码技术 - 公钥密码"
description: "用公钥加密，用私钥解密"
tags: [security, cryptography]
---

* 目录
{:toc}

## 1. 投币寄物柜的使用方法
首先，将物品放在寄物柜中。然后，投入硬币并拔出钥匙，就可以将寄物柜关闭了。关闭后的寄物柜，没有钥匙是无法打开的。

只要有硬币，任何人都可以打开寄物柜，但寄物柜一旦被关闭，再怎么投币也无法打开。打开寄物柜需要使用钥匙，而不是硬币。

因此我们可以说，硬币是`关闭寄物柜的密钥`，而钥匙则是`打开寄物柜的密钥`。

## 2. 密钥配送问题
### 2.1 什么是密钥配送问题
在现实世界中使用对称密码时，我们一定会遇到**密钥配送问题**（key distribution problem)。尽管一开始可能觉得这并不是什么大问题，但这个问题却是很难从根本上得到解决。

在通信过程中，如果发送密钥，则窃听者Eve也能够完成解密：
/************************** 图 *******************/

当然，如果Eve无法推测出通信中使用的是什么密码算法，那么即使得到密钥也是无法解密的。然而，密码算法本来就应该是以公开为前提的，隐蔽式安全性（security by obsecurity)是非常危险的。

`密钥必须要发送，但又不能发送`，这就是对称密码的密钥配送问题。

解决密钥配送问题的方法有以下几种。
* 通过事先共享密钥来解决
* 通过密钥分配中心来解决
* 通过Diffie-Hellman密钥交换来解决密钥配送问题
* 通过公钥密码来解决密钥配送问题

### 2.2 通过事先共享密钥来解决
密钥配送问题最简单地一种解决方法，就是事先用安全的方式将密钥交给对方，这称为密钥的**事先共享**。这可以说是一种理所当然的方法吧。

事先共享密钥尽管有效，但却有一定的局限。

首先，要想事先共享密钥，就需要一种安全的方式将密钥交给对方。如果是公司里坐在你旁边的同事，要事先共享密钥可能非常容易，只要将密钥保存在存储卡中交给他就可以。然而，要将密钥安全地交给一个前几天刚刚在网上认识的朋友就非常困难。如果用邮件等方式发送，则密钥可能会把窃听。另外邮寄存储卡也不安全，因为在邮寄的途中可能被别人窃取。

此外，即便能够实现实现共享密码，但在人数很多的情况下，通信所需要的密码数量也会增大，就产生了问题。例如，一个公司有1000名员工，则需要的密钥数量为：1000 * 999 / 2 = 499500。

这么大数量的密钥实在是不现实的。因此，事先共享密钥尽管有效，但依然有一定的局限性。

### 2.3 通过密钥分配中心来解决
如果所有参与加密通信的人都需要事先共享密钥，则密钥的数量会变得巨大，在这样的情况下，我们可以使用**密钥分配中心**（Key Distribution Center，KDC）来解决密钥配送问题。

当需要进行加密通信时，密钥中心会生成一个通信密钥，每个人只要和密钥分配中心事先共享密钥就可以了。

在公司中，我们配置一台充当密钥分配中心的计算机。这台计算机中有一个数据库，其中保存了Alice的密钥、Bob的密钥。。。等所有员工的密钥。也就是说，如果公司有1000名员工，那么数据库就会保存1000给密钥。

当有新员工入职时，密钥分配中心会为该员工生成一个新的密钥，并保存在数据库中。而新员工则会在入职时从密钥分配中心的计算机上领取自己的密钥，就像领取工作证一样。

这样一来，密钥分配中心就拥有了所有员工的密钥，而每个员工则拥有自己的密钥。那么，现在Alice再向Bob发送加密邮件时，就需要进行一下步骤。
1. Alice向密钥分配中心发出希望与Bob进行通信的请求。
2. 密钥分配中心通过伪随机数生成器生成一个会话密钥，这个密钥是供Alice与Bob在本次通信中使用的临时密钥。
3. 密钥分配中心从数据库中取出Alice的密钥和Bob的密钥。
4. 密钥分配中心用Alice的密钥对会话密钥进行加密，并发送给Alice。
5. 密钥分配中心用Bob的密钥对会话密钥进行加密，并发送给Bob。
6. Alice对来自密钥分配中心的会话密钥进行解密，得到会话密钥。
7. Alice用会话密钥对邮件进行加密，并将邮件发送给Bob。
8. Bob对来自密钥分配中心的会话密钥进行解密，得到会话密钥。
9. Bob用会话密钥对来自Alice的密文进行解密。
10. Alice和Bob删除会话密钥。

以上就是通过密钥分配中心完成Alice和Bob的通信的过程。

密钥分配中心尽管有效，但也有局限。首先，每当员工进行加密通信时，密钥分配中心计算机都需要进行上述处理。随着员工的数量增加，密钥分配中心的负荷也会随之增加。如果密钥分配中心计算机发生故障，全公司的加密通信就会瘫痪。此外，主动攻击者Mallory也可能会对密钥分配中心下手。如果Mallory入侵了密钥分配中心计算机，并盗取了密钥数据库，则后果会十分严重，因为全公司所有的加密通信都会被Mallory破译。因此，如果要使用密钥分配中心，就必须妥善处理上述问题。

### 2.4 通过Diffie-Hellman密钥交换来解决密钥配送问题
这里的交换，并不是指东西坏了需要换一个，而是指发送者和接受者之间相互传递信息的意思。

在Diffie-Hellman密钥交换中，进行加密通信的双方需要交换一些信息，而这些信息即使被Eve窃听也没有问题。

根据所交换的信息，双方可以各自生成相同的密钥，而Eve却无法生成相同的密钥。Eve虽然能够窃听到双方所交换的信息，但却无法根据这些信息生成和双方相同的密钥。关于Diffie-Hellman密钥交换，将在以后的文章中介绍。😄

### 2.5 通过公钥密码来解决密钥配送问题
在对称密码中，加密密钥和解密密钥是相同的，但公钥密码中，加密密钥和解密密钥确是不同的。只要拥有加密密钥，任何人都可以进行加密，但没有解密密钥是无法解密的。因此，公钥密码的一个重要性质，`就是只有拥有解密密钥的人才能进行解密`。

接收者事先将加密密钥发送给发送者，这个加密密钥即便被窃听者获取也没有问题。发送者使用加密密钥对通信内容进行加密并发送给接受者，而只有拥有解密密钥的人（即接收者本人）才能够进行解密。这样一来，就用不着将解密密钥配送给接收者了，也就是说，对称密钥的密钥配送问题，就可以通过公钥密码来解决。

## 3. 公钥密码
### 3.1 什么是公钥密码
**公钥密码**（public-key cryptography）中，密钥分为加密密钥和解密密钥两种。发送者用加密密钥对消息进行加密，接受者用解密密钥对密文进行解密。`加密密钥是发送者加密时使用的，而解密密钥是接受者解密时使用的。`

仔细思考一下加密密钥和解密密钥的区别，我们可以发现：
* 发送者只需要加密密钥
* 接受者只需要解密密码
* 解密密码不可以被窃听者获取
* 加密密码被窃听者获取也没问题

也就是说，解密密钥从一开始就由接受者自己保管的，因此只要将加密密钥发送给发送者就可以解决密钥配送问题了，而根本不需要配送解密密钥。

公钥密码中，加密密钥一般是公开的。正是由于加密密钥可以任意公开，因此该密钥被称为**公钥**。相当的，`解密密钥是绝对不能公开，`这个密钥只能由自己来使用，因此称为**私钥**。

公钥和私钥是一一对应的，一对公钥和私钥统称为密钥对(key pair)。有公钥进行加密的密文，必须使用与该公钥配对的私钥才能够解密。密钥对中的两个密钥之间具有非常密切的关系-数学上的关系-因此公钥和私钥是不能单独生成的。

### 3.2 公钥密码的历史
[公钥密码的历史](http://baike.baidu.com/item/%E5%85%AC%E9%92%A5#1)

### 3.3 公钥通信流程
参考下面的通信流程图，看一看在Alice和Bob之间到底传输了哪些信息。其实他们之间所传输的信息只有两个：Bob的公钥以及用Bob的公钥加密的密文。由于Bob的私钥没有出现在通信内容中，因此窃听者Eve无法对密文进行解密。


/********************* 图 *****************/

### 3.4 各种术语
公钥密码又称**非对称密码**(asymmetric cryptography)。
私钥又称**个人密钥、私有密钥、非公开密钥、秘密密钥**。

### 3.5 公钥密码无法解决的问题
公钥密码解决了密钥配送问题，但这并不意味着它能解决所有的问题，因为我们需要判断所得到的公钥是否正确合法，这个问题被称为`公钥认证问题`。此外，还有一个问题，`它的处理速度只有对称密码的几百分之一`。

## 4. 时钟运算
RSA的数学理论基础。

## 5. RSA
RSA -- 使用最广泛的公钥密码算法。
### 5.1 什么是RSA

### 5.2 RSA 加密

### 5.3 RSA 解密

### 5.4 生成密钥对

### 5.5 具体实践一下

## 6. 对RSA的攻击

## 7. 其他公钥密码

## 8. 关于公钥密码的Q & A

