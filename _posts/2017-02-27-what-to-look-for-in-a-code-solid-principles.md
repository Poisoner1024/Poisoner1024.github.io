---
layout: post
title: "在代码审查中我们究竟想发现什么？- SOLID原则篇"
description: "面向对象编程的五大核心原则"
tags: [web, develop, code]
---
这是关于`在代码审查中我们究竟想发现什么?`系列6篇文章中的第5篇。

### 1. 什么是SOLID？
*SOLID*原则是面向对象设计与编程的五个核心原则。这篇博文并不要是告诉你这是原则是什么或者深入讨论你为什么要遵循它们，而是要帮助Code Reviewer来更容易的发现当代码没有遵循这些原则的时候的问题。

*SOLID* 代表：
> * **S** - 单一只能原则
> * **O** - 开-闭原则
> * **L** - 里氏替换原则
> * **I** - 接口分离原则
> * **D** - 依赖反转原则


### 2. 单一只能原则（Single Responsibility Principle）
> 不要多于一个原因修改类。
> 
> There should never be more than one reson for a class to change.

很多时候一个很难从单一的一个code review中就能发现。按照定义，代码作者只应有有单一的原因来修改基础代码，如bug fix, 新功能后者一个专门的重构。

你想看看在类中的哪些方法很可能在同一时间被改变，哪一系列方法在修改其他方法的时候不太可能会被改变。例如：
{% include image.html path="documentation/what-to-look-for-in-a-code-review/CR5SRP1.png" path-detail="documentation/what-to-look-for-in-a-code-review/CR5SRP1.png" alt="CR5SRP1" %}
从代码的比对中可以看到，新增了一个功能在*TweetMonitor*里面，在某类用户接口中使其能够在leaderboard画出前10位Tweeters。这个看起来比较合理，因为它用通过*onMessage*方法聚合的数据，这里有违反SRP的迹象。方法*onMessage*和*getTweetMessageFromFullTweet*都是关于接收和解析一个Twitter信息的，然后*draw*是将所有的用来在UI上显示的数据进行重新组织。

Reviewer需要指出这两个职责，找出更好的办法来分离这些功能：可以通过把Twitter字符串解析移到不同的类中，或者创建一个新类来渲染leaderboard。


### 3. 开-闭原则（Open-Closed Principle）
> 软件实体应该是对扩展开放，对修改关闭。
> 
> Software entities should be open for extension, but closed for modification.

当你看到一系列的*if*表达式来检查某个特定的类型，你可能发现了违反这一原则的线索。
{% include image.html path="documentation/what-to-look-for-in-a-code-review/CR5OCP1.png" path-detail="documentation/what-to-look-for-in-a-code-review/CR5OCP1.png" alt="CR5OCP1" %}
上面的代码，当有一个新的*Event*类型添加到系统的时候，新类型的创建者可能就需要添加另外一个*else*来处理这个新事件类型。

多态（polymorphism）是用来移除*if*的好的选择：
{% include image.html path="documentation/what-to-look-for-in-a-code-review/CR5-OCP-remove-if.png" path-detail="documentation/what-to-look-for-in-a-code-review/CR5-OCP-remove-if.png" alt="CR5-OCP-remove-if" %}

{% include image.html path="documentation/what-to-look-for-in-a-code-review/CR5-OCP3.png" path-detail="documentation/what-to-look-for-in-a-code-review/CR5-OCP3.png" alt="CR5-OCP3" %}
同样，对这个问题也不止这一个解决方案，但是关键是移除复杂的 *if/else*和*instanceof*检查。

### 4. 里氏替换原则（Liskov Substitution Principle）
> 能对基础类引用的方法必须能够在不知道引申类对象的情况下使用它。
> 
> Functions that use references to base classes must be able to use objects of derived classes without knowing it.

一个很容易发现违法这一原则的方法是寻找明确的转型（casting）。如果你必须把一个对象转换成其他类型，在不了解引申类的情况下没有使用基类。
更多细小违法这一原则的迹象可以再检查下面的内容里发现。
* 前提条件不能强加子类型；
* 后置调价不能弱化为子类型；
想象下，例如，我们有个抽象的*Order*，它有一系列的子类，*BookOrder*，*ElectronicsOrder*,等等。 方法*placeOrder*可以带有一个*Warehouse*，并且可以用它来修改仓库中的物理原件的目录等级。
{% include image.html path="documentation/what-to-look-for-in-a-code-review/CR5-LSP4.png" path-detail="documentation/what-to-look-for-in-a-code-review/CR5-LSP4.png" alt="CR5-LSP4" %}

现在我们引入电子礼品卡的概念， 它只是仅仅往钱包中放入收支，但是不需要物理目录。如果实现如*GiftCardOrder*，那么*placeOrder*方法不需要使用*warehouse*参数。
{% include image.html path="documentation/what-to-look-for-in-a-code-review/CR5-LSP12.png" path-detail="documentation/what-to-look-for-in-a-code-review/CR5-LSP12.png" alt="CR5-LSP12" %}
这个看上去像逻辑上使用了继承（inheritance），但事实上你可以认为*GiftCardOrder*中的代码期望这个类能与其他的类有相似的功能，如，你可以把这个传给所有的子类：
{% include image.html path="documentation/what-to-look-for-in-a-code-review/CR5-LSP2.png" path-detail="documentation/what-to-look-for-in-a-code-review/CR5-LSP2.png" alt="CR5-LSP2" %}
但是在*GiftCardOrder*里面这个不会被传递，*GiftCardOrder*有一个不同的订单行为。如果你看到这样的代码，你需要注意这里的继承的使用，可能订单行为可以插在用组合代替继承的实现里面。

### 5. 接口分离原则（Interface Segregation Principle）
> 许多特定客户端的接口要好于一个通用目的的接口
> 
> Many client specific interfaces are better than one general purpose interface

如果接口里面有许多的方法，你可能很容易发现这一远程被违反了。这个原则也是遵从SRP的，你可以看到有多个方法的接口实际上在一个区域功能上又多个职责。

但是有时一个接口甚至只有区区两个方法也应该别分成两个接口：
{% include image.html path="documentation/what-to-look-for-in-a-code-review/CR5-LSP3.png" path-detail="documentation/what-to-look-for-in-a-code-review/CR5-LSP3.png" alt="CR5-LSP3" %}
这个例子里面，如果*decode*方法有时不需要，在不同的使用场景下，codec可能被当作一个encoder或者decoder，这里最好将*SimpleCodec*接口分开成*Encoder*和*Decoder*。有些类可以选择两个都实现，但是它可以不用实现一个*override*的方法当它不需要的时候，或者一个类只需要*Encoder*就不需要知道有*decode*的实现。

### 6. 依赖反转原则（Dependency Inversion Principle）
> 依赖于抽象，不要依赖于具体。
> 
> Depend upon Abstractions. Do not depend upon concretions.

我们尝试着去找到一些违反这一原则的简单的cases，比如，*liberal*中用*new*关键字（代替依赖注入或工厂）和十分容易接受的集合类型（如申明*ArrayList*变量或参数来替代*List*），reviewer要确定代码作者使用或新建了正确的抽象。

看个例子，service-level代码使用了一个直接的*database*连接来进行读写数据操作：
{% include image.html path="documentation/what-to-look-for-in-a-code-review/CR5-DIP1.png" path-detail="documentation/what-to-look-for-in-a-code-review/CR5-DIP1.png" alt="CR5-DIP1" %}
这段代码依赖于很多特殊的实现细节：*JDBC*作为关系型*database*连接；具体的*database*的*SQL*；已知的*database*结构；等等。这个确实需要存在你的系统的某个位置，但是不应该出现在这里，因为这里的其他方法并不需要了解关于*database*的细节。做好是提取出*DAO*或者是使用*Repository Pattern*,往这个*service*里面注入*DAO*或*repository*。

### 7. 写在最后
有些代码的坏味道会反映出一个或多个SOLID的原则被违反了：
* 长的*if/esle*申明
* 向下转型
* 很多公共方法
* 抛出*UnsupportedOperatioException*的实现方法

和所有的设计问题一样，需要找到团队的喜好与遵循这些原则之间的平衡。但是如果你在code review中发现复杂的代码，你可能会发现这时如果你应用上述的一个原则会提供一个更简单的，更容易理解的解决方案。




