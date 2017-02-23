---
layout: post
title: "大型网站技术架构"
description: "网站架构其实并不难，真正能解决问题的技术一定是简单的。"
tags: [web, develop]
---
作为一名软件工程师，我们再解决问题之前，很有必要去认真思考面对的问题究竟是什么？有哪些技术方案可以选择？其基本原理是什么？只有弄清楚这些个问题，才能从根本上去解决问题。同样对于今天随处可以触及的网站问题也是这样，想做一名与web相关的技术人员，就不得不把网站的技术架构摸清楚，以不至于一遇到稍微有难度的问题后，就只会在Google/度娘上一顿乱Search,沦为编码是否成功全部靠运气的码农🙏！

### 1. 大型网站软件系统的特点
什么是大型网站？可能很难用一个标准去定义，这时候我们可以从它与传统企业应用系统相比，看出大型互联网应用系统有一下特点：
* `高并发，大流量`：需要面对高并发用户，大流量访问。Google日均PV数35亿，日均IP访问数3亿；腾讯QQ的最大在线用户数1.4亿（2011年数据）；淘宝2012年“双11”活动一天交易额超过191亿，活动开始第一分钟独立访问用户达到1000万。
* `高可用`：系统7*24小时不间断服务。大型互联网网站的宕机时间通常会成为新闻焦点，例如2010年百度域名被黑客劫持导致不能访问，成为重大新闻热点。
* `海量数据`：需要存储、管理海量数据，需要使用大量的服务器。Facebook每周上传的照片数目接近10亿，百度收录的网页数目有数百亿，Google有近百万台服务器为全球用户提供服务。
* `用户分布广泛，网络情况复杂`：许多大型互联网都是为全球用户提供服务的，用户分布广泛，各地网络情况千差万别。在国内，还有各个运营商网络互通难的问题。而中美光缆数次故障，也让一些对国外用户依赖较大的网站不得不考虑在海外建立数据中心。
* `安全环境恶劣`：由于互联网的开放性，使得互联网网站更容易受到攻击，大型网站几乎每天都会被黑客攻击。2011年国内多个重要网站泄密用户密码，让普通用户也直面一次互联网安全问题。
* `需求快速变更，发布频繁`：和传统软件的版本发布频率不同，互联网产品为快速适应市场，满足用户需求，其产品发布频率极高。Office的产品版本以年为单位发布，而一般大型网站的产品每周都有新版本发布上线，至于中小型网站的发布就更频繁了，有时候一天会发布几十次。
* `渐进式发展`：与传统软件产品或者企业应用系统一开始就规划好全部的功能和非功能需求不同，几乎所有的大型互联网网站都是从一个小网站开始，渐进地发展起来的。Facebook是扎克伯克在哈佛大学的宿舍里开发的；Google的第一台服务器部署在斯坦福大学的实验室里；阿里巴巴则是在马云家的客厅里诞生的。好的互联网产品都是慢慢运营出来的，不是一开始就开发好的，这也正好与网站架构的发展演化过程对应的。 

### 2. 大型网站架构演化发展历程
大型网站的技术挑战主要来自于庞大的用户，高并发的访问和海量的数据，任何简单的业务一旦需要处理数以P计的数据和面对数以亿计的用户，问题就会变得很棘手。大型网站架构主要即使解决这类问题，下面我们来看一个大型的网站是如何成长起来的？
#### 2.1 初始阶段的网站架构
{% highlight markdown %}
* 小型网站最开始时没有太多人访问，只需要一台服务器就绰绰有余。
{% endhighlight %}

{% include image.html path="documentation/web-site-architecture/big-web-site-stage1.png" path-detail="documentation/web-site-architecture/big-web-site-stage1.png" alt="初始阶段" %}

应用程序、数据库、文件等所有的资源都在一台服务器上。通常服务器操作系统用Linux，应用程序用PHP开发，然后部署在Apache上，数据库使用MySQL，汇聚各种免费开源软件及一台廉价服务器就可以开始网站的发展之路了。


#### 2.2 应用服务和数据服务分离阶段的网站架构
{% highlight markdown %}
随之网站业务的发展，一台服务器逐渐不能满足需求：
* 越来越多的用户访问导致性能越来越差-并发处理能力差；
* 越来越多的数据导致存储空间不足；
这时就需要将应用和数据分离。
{% endhighlight %}

{% include image.html path="documentation/web-site-architecture/big-web-site-stage2.png" path-detail="documentation/web-site-architecture/big-web-site-stage2.png" alt="应用服务和数据服务分离" %}

此时整个网站使用三台服务器：应用服务器，文件服务器和数据库服务器，三台服务器对硬件资源要求各不相同：应用服务器需要处理大量的业务逻辑，因此需要更快更强大的CPU，数据库服务器需要快速磁盘检索和数据缓存，因此需要更快的硬盘和更大的内存，文件服务器需要存储大量的用户上传文件，因此需要更大的硬盘。

#### 2.3 使用缓存改善网站性能阶段的网站架构
{% highlight markdown %}
用户持续增多：
* 数据库压力太大导致访问延迟；
* 网站性能差，用户体验收到影响；
网站访问特点同样遵循二八定律：80%的业务访问集中在20%的数据上。
数据缓存方案应运而生。
{% endhighlight %}

{% include image.html path="documentation/web-site-architecture/big-web-site-stage3.png" path-detail="documentation/web-site-architecture/big-web-site-stage3.png" alt="使用缓存改善网站性能" %}

网站使用的缓存可以分为两种：缓存在应用服务器的本地缓存和缓存在专门的分布式缓存服务器上的远程缓存。本地缓存的访问速度更快一些，但是受应用服务器内存限制，缓存数据量有限。远程分布式缓存可以使用集群的方式，部署大内存的服务器作为专门的缓存服务器，可以在理论上做到不受内存容量的限制。

#### 2.4 使用应用服务器集群改善网站的并发能力阶段的网站架构
{% highlight markdown %}
当使用缓存将数据访问压力得到有效化解后，单一应用服务器能够
处理的请求连接有限：
* 网站访问高峰期，应用服务器成为整个网站的瓶颈；
使用集群是网站解决高并发、海量数据问题的常用手段。
{% endhighlight %}

{% include image.html path="documentation/web-site-architecture/big-web-site-stage4.png" path-detail="documentation/web-site-architecture/big-web-site-stage4.png" alt="使用应用服务器集群改善网站的并发能力" %}

<p>当有一台服务器的处理能力、存储空间不足时，不要企图去更换更强大的服务器，对于大型网站而言，不管多么强大的服务器，都满足不了网站持续增长的业务需求。此时，应该考虑增加一台服务器分担原有服务器的访问以及存储压力。</p>
<p>对网站架构而言，只要能通过增加一台服务器的方式改善负载压力，就可以以同样的方式持续增加服务器不断改善系统性能，从而式样系统的可伸缩性。应用服务器实现集群是网站可伸缩集群架构设计中较为简单成熟的一种方式。</p>
<p>通过负载均衡调度服务器，可将来自用户浏览器的访问请求分发到应用服务器集群中的任何一台服务器上，如果有更多的用户，就在集群中加入更多的应用服务器，是应用服务器不在成为整个网站的瓶颈。</p>

#### 2.5 数据库读写分离阶段的网站架构
{% include image.html path="documentation/web-site-architecture/big-web-site-stage5.png" path-detail="documentation/web-site-architecture/big-web-site-stage5.png" alt="数据库读写分离" %}

#### 2.6 使用反向代理和CDN加速网站响应阶段的网站架构
{% include image.html path="documentation/web-site-architecture/big-web-site-stage6.png" path-detail="documentation/web-site-architecture/big-web-site-stage6.png" alt="使用反向代理和CDN加速网站响应" %}

#### 2.7 使用分布式文件系统和分布式数据库系统阶段的网站架构
{% include image.html path="documentation/web-site-architecture/big-web-site-stage7.png" path-detail="documentation/web-site-architecture/big-web-site-stage7.png" alt="使用分布式文件系统和分布式数据库系统" %}

#### 2.8 使用NoSQL和搜索引擎阶段的网站架构
{% include image.html path="documentation/web-site-architecture/big-web-site-stage8.png" path-detail="documentation/web-site-architecture/big-web-site-stage8.png" alt="使用NoSQL和搜索引擎" %}

#### 2.9 业务拆分阶段的网站架构
{% include image.html path="documentation/web-site-architecture/big-web-site-stage9.png" path-detail="documentation/web-site-architecture/big-web-site-stage9.png" alt="业务拆分" %}

#### 2.10 分布式服务阶段的网站架构
{% include image.html path="documentation/web-site-architecture/big-web-site-stage10.png" path-detail="documentation/web-site-architecture/big-web-site-stage10.png" alt="分布式服务" %}

### 3. 大型网站架构技术一览

### 4. 写在最后
本人也算是生在互联网的时代，接触了各式各样的网站，同时自己也开发与维护网站，在遇到Taobao李智慧老师的这本《大型网站技术架构》之前也一直希望能弄明白网站架构到底是怎么一回事，仅以此文致敬作者。