---
layout: post
title: "Richardson成熟度模型"
description: "RESTFul API成熟度模型"
tags: [web, service, rest]
---
* 目录
{:toc}

原文：[Richardson Maturity Model](https://martinfowler.com/articles/richardsonMaturityModel.html)

### 通往REST的荣耀之巅的阶梯
Leonard Richardson开发了一个模型，将通往REST的方式划分非三个步骤。它们分别是资源，HTTP的动词，和超媒体控件。这个模型非常好的解释了如何使用REST中的技术。一图胜千言：
{% include image.html path="documentation/rest/maturity-model-overview.png" path-detail="documentation/rest/maturity-model-overview.png" alt="maturity-model-overview" %}

### Level 0 - the Swamp of POX
模型的起始点是只把HTTP作为与远程交互的传输系统，并没有使用任何关于它在Web中的机制。至关重要的是，在这里所做的只是把HTTP当做你自己的远程交互机制中的通道机制，一般情况下这个是基于远程过程调用的（Remote Procedure Invocation）。上图

{% include image.html path="documentation/rest/level0-example.png" path-detail="documentation/rest/level0-example.png" alt="level0-example" %}

如上图所示，假设我想在我的医生那里挂个号。我的挂号软件首先需要知道我的医生在对应的日子里什么时间段有空，于是就向医院的挂号系统发个请求来获取这个信息。在Level 0的场景下，医院会在某些URI下暴露一个service endpoint。那么我就可以通过POST方式发一个包含我的请求的详细内容到这个endpoint。
{% highlight html %}
POST /appointmentService HTTP/1.1
[various other headers]

<openSlotRequest date ="2010-01-04" doctor ="mjones"/>
{% endhighlight %}

服务器返回一个包含这些信息的文档：
{% highlight html %}
HTTP/1.1 200 OK
[various headers]

<openSlotList>
  <slot start ="1400" end ="1450">
    <doctor id ="mjones"/>
  </slot>
  <slot start =1600" end ="1650">
      <doctor id ="mjones"/>
  </slot>
</openSlotList>
{% endhighlight %}

在这个例子里面我用的是XML，其实这个内容实际上可以是其他类型：JSON, YAML, key-value pairs, 或者其他定制的格式。

于是下一步就是挂号了，我可以再次通过POST往对应的endpoint发个请求。
{% highlight html %}
POST /appointmentService HTTP/1.1
[various other headers]

<appointmentRequest>
  <slot doctor ="mjones" start ="1400" end ="1450"/>
  <patient id ="jsmith"/>
</appointmentRequest>
{% endhighlight %}

如果没什么问题的话，我能得到一个挂号成功的响应。
{% highlight html %}
HTTP/1.1 200 OK
[various headers]

<appointment>
  <slot doctor = "mjones" start = "1400" end = "1450"/>
  <patient id = "jsmith"/>
</appointment>
{% endhighlight %}

如果问题的话，其他人在我之前把这个号挂了，那么我能在返回的消息体里面得到这类的错误信息。
{% highlight html %}
HTTP/1.1 200 OK
[various headers]

<appointmentRequestFailure>
  <slot doctor ="mjones" start ="1400" end ="1450"/>
  <patient id ="jsmith"/>
  <reason>Slot not available</reason>
</appointmentRequestFailure>
{% endhighlight %}
到目前为止，这是一个直接了当的RPC风格的系统。它只是简单的把POX（plain old XML）进行来回传送。如果你使用SOAP或者XML-RPC,基本上市一样的机制，唯一不同的是你把XML信息包装是某些类型的信封里面。

### Level 1 - Resources
在RMM(Resource Monitoring & Managment)中，通往REST荣耀之巅的第一步是进入资源的概念。所以现在不是将我们的所有请求发送到一个单一的service endpoint，我们现在开始与独立的资源进行对话。上图：
{% include image.html path="documentation/rest/level1-example.png" path-detail="documentation/rest/level1-example.png" alt="level1-example" %}

按照我们一开始的请求，我们可能有专门针对特定医生的一个资源。
{% highlight html %}
POST /doctors/mjones HTTP/1.1
[various other headers]

<openSlotRequest date ="2010-01-04"/>
{% endhighlight %}
回复的消息中带有一样的基本信息，但是每一个时间段现在是一个资源，可以独立的发放。
{% highlight html %}
HTTP/1.1 200 OK
[various headers]


<openSlotList>
  <slot id ="1234" doctor ="mjones" start ="1400" end ="1450"/>
  <slot id ="5678" doctor ="mjones" start ="1600" end ="1650"/>
</openSlotList>
{% endhighlight %}

用特定的资源去挂号意味着发送请求到特定的时间段。
{% highlight html %}
POST /slots/1234 HTTP/1.1
[various other headers]

<appointmentRequest>
  <patient id ="jsmith"/>
</appointmentRequest>
{% endhighlight %}

如果一切正常的话，我可以得到一个与之前相识的回复。
{% highlight html %}
HTTP/1.1 200 OK
[various headers]

<appointment>
  <slot id ="1234" doctor ="mjones" start ="1400" end ="1450"/>
  <patient id ="jsmith"/>
</appointment>
{% endhighlight %}

现在的不同之处是，如果任何人想做任何与挂号相关的事情，比如预约检查，它们首先要得到一个预约资源，这个资源应该由一个URI，如：http://royalhope.nhs.uk/slots/1234/appointment，然后往这个资源发请求。

对想玩这样的喜好对象的人来说，这个就像对象身份的概念。不是通过传递参数调用一些函数，我们通过请求某个特定对象，这个对象为获得其他信息提供了参数。

### Level 2 - HTTP Verbs
在这里我的所有交互请求中，我使用了HTTP中的POST动词，但是一些人使用GETs来替代，或者用其他的。在level 0和1的这些层面，这并无多大区别，它们都是被用来当做通道机制，允许你在HTTP之上完成你的交互请求。Level 2中移除了这个，尽可能按照HTTP自身的使用惯例来使用HTTP的动词。上图：
{% include image.html path="documentation/rest/level2-example.png" path-detail="documentation/rest/level2-example.png" alt="level2-example" %}

对于我们的时间段的列表，这个意味着我们相用GET。
{% highlight html %}
GET /doctors/mjones/slots?date=20100104&status=open HTTP/1.1
Host: royalhope.nhs.uk
{% endhighlight %}

这个的返回内容应该是和POST一样的。
{% highlight html %}
HTTP/1.1 200 OK
[various headers]

<openSlotList>
  <slot id ="1234" doctor ="mjones" start ="1400" end ="1450"/>
  <slot id ="5678" doctor ="mjones" start ="1600" end ="1650"/>
</openSlotList>
{% endhighlight %}
在Level 2，像这样使用GET来请求非常的关键。HTTP定义GET是一个安全的操作，GET请求不会对任何东西的状态造成任何重要的改变。这样我们就能安全的引用GETs,无论多少次，也无论什么顺序，每一次都能得到相同的结果。这个的一个重要的影响就是GET允许任何请求子程序的参与者可以使用**缓存**，这是web能提供好的性能的关键因素。HTTP有各种方式来支持缓存，可以被连接中的所有参与者使用。通过遵循HTTP的这些规则，我们可以利用这个优势。

我们需要一个确实会改变状态的HTTP动词，POST或PUT，来完成挂号操作。我将用我之前用的一样的POST请求。
{% highlight html %}
POST /slots/1234 HTTP/1.1
[various other headers]

<appointmentRequest>
  <patient id ="jsmith"/>
</appointmentRequest>
{% endhighlight %}

这个将不再过多的讨论是用POST好，还是PUT好。当时我确实想指出，一些人错误的给POST/PUT与create/update做了对应。它们之间的选择与选择create还是update不同。

尽管我在level 1中同样用了POST，这里还有一个重要的不同点在远程服务如何响应中。如果一切正常的话，服务将返回一个201的响应码来表示新建了一个资源。
{% highlight html %}
HTTP/1.1 201 Created
Location: slots/1234/appointment
[various headers]

<appointment>
  <slot id ="1234" doctor ="mjones" start ="1400" end ="1450"/>
  <patient id ="jsmith"/>
</appointment>
{% endhighlight %}
201的响应包含一个带有位置属性的URI，客户可以拿着这个URI来进一步获取资源当前的状态。这里的响应也包含一个资源的表示来告诉客户立即做一个额外的请求。

如果出错了，这里是另一个不同点，比如订上这个时间段的另有其人：
{% highlight html %}
HTTP/1.1 409 Conflict
[various headers]

<openSlotList>
  <slot id ="5678" doctor ="mjones" start ="1600" end ="1650"/>
</openSlotList>
{% endhighlight %}
这个请求的重要一部分是用HTTP响应码来指示出错了。这里的情况是，409看上去是在不兼容的方式下指示有其他人已经更新了资源的好的选择。而不是使用返回200，然后包含一个错误信息。在level 2中我们明确的使用一些像这样的错误响应。由协议的设计者来决定改使用什么代码，但是当错误发生时，这里应该用非200系列的响应。

这里有个不一致的问题。REST的拥护者说要使用所有的HTTP动词。他们还说REST正在尝试从web的实践中的成功中得到学习来证明他们的方法。但是在实践中，万维网并不用PUT或DELET。这里有合理的理由来支持更多的用PUT和DELETE，但是web的已经证明的成功案例不在其中。

由Web中已经纯在的东西支持的关键是一定要将安全的（eg GET）与不安全的操作分开，将状态码与操作放在一起使用过来解决你在通信中遇到的各种各样的错误。

### Level 3 - Hypermedia Controls
终极level引入了一些你经常听到的关于那个丑陋的缩写：HATEOAS（Hypertext As The Engine Of Application State)。它解决了如果从一个开发的时间段列表中获知做什么来挂号的问题。上图：
{% include image.html path="documentation/rest/level3-example.png" path-detail="documentation/rest/level3-example.png" alt="level3-example" %}

我们从在level 2 中发送的一样的GET开始：
{% highlight html %}
GET /doctors/mjones/slots?date=20100104&status=open HTTP/1.1
Host: royalhope.nhs.uk
But the response has a new element

HTTP/1.1 200 OK
[various headers]

<openSlotList>
  <slot id ="1234" doctor ="mjones" start ="1400" end ="1450">
     <link rel ="/linkrels/slot/book" 
           uri ="/slots/1234"/>
  </slot>
  <slot id ="5678" doctor ="mjones" start ="1600" end ="1650">
     <link rel ="/linkrels/slot/book" 
           uri ="/slots/5678"/>
  </slot>
</openSlotList>
{% endhighlight %}
每一个时间段有一个链接，包含了URI来告诉我们如何预约。超媒体控件的关键点是它们告诉我们下一个该做什么，和我们需要对其进行操作的资源的URI。而不是我们必须知道往哪儿去post我们的预约请求，响应中的超媒体控件告诉我们如何对它进行操作。

仍然可以用level 2 中的POST：
{% highlight html %}
POST /slots/1234 HTTP/1.1
[various other headers]

<appointmentRequest>
  <patient id ="jsmith"/>
</appointmentRequest>
{% endhighlight %}

回复中包含了一些下一步做不同事情的超媒体控件。
{% highlight html %}
HTTP/1.1 201 Created
Location: http://royalhope.nhs.uk/slots/1234/appointment
[various headers]

<appointment>
  <slot id ="1234" doctor ="mjones" start ="1400" end ="1450"/>
  <patient id ="jsmith"/>
  <link rel ="/linkrels/appointment/cancel"
        uri ="/slots/1234/appointment"/>
  <link rel ="/linkrels/appointment/addTest"
        uri ="/slots/1234/appointment/tests"/>
  <link rel ="self"
        uri ="/slots/1234/appointment"/>
  <link rel ="/linkrels/appointment/changeTime"
        uri ="/doctors/mjones/slots?date=20100104@status=open"/>
  <link rel ="/linkrels/appointment/updateContactInfo"
        uri ="/patients/jsmith/contactInfo"/>
  <link rel ="/linkrels/help"
        uri ="/help/appointment"/>
</appointment>
{% endhighlight %}

超媒体控件的一个明显的好处是它允许服务器修改它的URI scheme而不对客户造成影响。只要客户查找"addTest"链接URI，那么服务器团队可以随意修改这些URIs而不是只有一个初始的入口。

还有一个进一步的好处，它可以帮助客户开发人员探索协议。这些链接给客户开发人员一个关于下一步该是什么的暗示。它没有给出所有信息："latest" 和 "cancel" 控件都指向相同URI - 它们需要搞明白一个是GET，另一个DELETE。但至少它给了他们一个开始点来思考更多信息是什么，以及在协议文档里面查看相似的URI。

相似的是，它允许服务提供团队可以发布新链接而不影响客户。如果客户端开发人员注意一下那些不太了解的链接，这些链接可以帮助他们对服务端做进一步探索。

这里没有觉得的标准关于如何表示超媒体控件。我这里所做的是使用REST in Practice里面流行的建议，这些建议是符合ATOM（RFC 4287）规范的。在目标URI属性里面用<link>元素，用<rel>属性来描述这类关系。

一种众所周知的直白的关系是 -- 任何服务端具体的服务就是一个完全合格的URI。ATOM表述了一个大家都知道linkrels的定义：链接关系的注册。就像我写得这些就是紧密的围绕着ATOM的要求，这也是level 3里面的首要任务。

### 这些层次的意义
我要强调一下，RMM，虽然是思考REST里的元素的一个很好的方式，但是它并不是REST自身层次的一个定义。Roy Fielding有过一个澄清说到level 3 RMM是REST的前提条件。像许多软件中的术语一样，REST有很多的定义，当是自从Roy Fielding创造了这个术语，他对它的定义应该比其他人的更具有意义和价值。

我发现这个RMM中的有用的东西是它提供了一个很好的一步一步的方式去理解隐含在restful思考方式后面的一些基本思想。依我看它就像一个工具，可以帮助我们学习REST中的概念，并不是某些应该被用在某种评估机制中的东西。我不认为我们已经有了足够多的例子来说明restful的方式对基础系统来说就一定是正确的。我确实认为它是个非常有吸引力的方式并且我在大多数场景下都建议的一种方式。

与Ian Robinson谈到这些时，他强调了一些他发现的关于这个模型很有吸引力的地方，当Richardson第一次表达RMM是一种与通用设计技术的关系。
* Level 1 是通过分而治之的方式处理复杂度的问题，把大的service endpoint分割成多个资源。
* Level 2 引用了一系列标准的verbs,这样我们可用用相同的方式处理相识的场景，移除了不变要的变化。
* Level 3 引入了可发现性，提供了一种让协议能够自描述的方法。

结果就是一个模型可以帮助我们思考我们想要提供一种什么样的HTTP服务，以及构建了人们希望和其进行交互的期望。


