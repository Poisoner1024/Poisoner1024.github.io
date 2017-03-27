---
layout: post
title: "设计模式 - 行为模式 - 策略"
description: "定义一系列算法，把它们一个个封装起来，并且使它们可以相互替换"
tags: [design,pattern]
---
### 对象行为型模式 - 策略（STRATEGY）

#### 1. 意图
定义一系列算法，把它们一个个封装起来，并且使它们可以相互替换。本模式使得算法可独立于使用它的客户而变化。

#### 2. 别名
政策（Policy)

#### 3. 动机
有许多算法可对一个正文流进行分析。将这些算法硬编码进使用它们的类中是不可取的，其原因如下：
* 需要换行功能的客户程序如果直接包含换行算法代码的话将会变得复杂，这使得客户程序庞大并且难以维护，尤其当其需要支持多种换行算法时问题会更加严重。
* 不同的时候需要不同的算法，我们不想支持我们并不使用的换行算法。
* 当换行功能是客户程序的一个难以分割的成分时，增加新的换行算法或改变现有算法将十分困难。

我们可以定义一些类来封装不同的换行算法，从而避免这些问题。一个以这种方法封装的算法称之为一个**策略（Strategy）**，如下图所示：
{% include image.html path="documentation/design-pattern/Strategy-Demo.png" path-detail="documentation/design-pattern/Strategy-Demo.png" alt="Strategy-Demo" %}
假设一个Composition类负责维护和更新一个正文浏览程序中显示的正文换行。换行策略不是Composition实现的，而是由抽象的Compositor类的子类各自独立地实现的。Compositor各个子类实现不同的换行策略。Composition维护对Compositor对象的一个引用。一旦Composition重新格式化它的正文，它就想这个职责转发给它的Compositor对象。Composition的客户指定应该使用哪一种Compositor的方式是直接将它想要的Compositor装入Composition中。

#### 4. 适用性
* 许多相关的类仅仅是行为有异。“策略”提供了一种用多个行为中的一个行为来配置一个类的方法。
* 需要使用一个算法的不同变体。例如，你可能会定义一些反映不同的空间/时间权衡的算法。当这些变体实现为一个算法的类层次时，可以使用策略算法。
* 算法使用客户不应该知道的数据。可使用策略模式以避免暴露复杂的、与算法相关的数据结构。
* 一个类定义了多种行为，并且这些行为在这个类的操作中以多个条件语句的形式出现。将相关的条件分支移入它们各自的Strategy类中以代替这些条件语句。

#### 5. 结构
{% include image.html path="documentation/design-pattern/Strategy-Pattern-Structure.png" path-detail="documentation/design-pattern/Strategy-Pattern-Structure.png" alt="Strategy-Pattern-Structure" %}

#### 6. 参与者
* Strategy（策略，如Compositor)
    - 定义所有支持的算法的公共接口。Context使用这个接口来调用某ConcreteStrategy定义的算法。
* ConcrateStrategy（具体策略，如SimpleCompositor)
    - 以Strategy接口实现某具体算法。
* Context（上下文，如Composition）
    - 用一个ConcreteStrategy对象来配置。
    - 维护一个对Strategy对象的引用。
    - 可定义一个接口来让Strategy访问它的数据


























