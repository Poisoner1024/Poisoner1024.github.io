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

#### 7. 协作
* Strategy和Context相互作用以实现选定的算法。当算法被调用时，Context可以将该算法所需要的所有数据都传递给该Strategy。或者，Context可以将自身作为一个参数传递给Strategy操作。这就让Strategy在需要时可以回调Context。
* Context将它的客户的请求转发给它的Strategy。客户通常创建并传递一个ConcreteStrategy对象给该Context；这样，客户仅与Context交互。通常有一系列的ConcreteStrategy类可供客户从中选择。

#### 8. 效果
Strategy模式有下面一些优点和缺点：
* `相关算法系列`  Strategy类层次为Context定义了一系列的可供重用的算法或行为。继承有助于析取这些算法中的公共功能。

* `一个替代继承的方法`  继承提供了另一种支持多种算法或行为的方法。你可以直接生成一个Context类的子类，从而给它以不同的行为。但这会将行为硬性编制到Context中，而将算法的实现与Context的实现混合起来，从而使Context难以理解、难以维护和难以扩展，而且还不能动态地改变算法。最后你得到一堆相关的类，它们之间的唯一差别是它们所使用的算法或行为。将算法封装在独立的Strategy类中使得你可以独立于其Context改变它，使它易于切换、易于理解、易于扩展。

* `消除了一些条件语句`  Strategy模式提供了用条件语句选择所需的行为以外的另一种选择。当不同的行为堆彻在一个类中时，很难避免使用条件语句来选择合适的行为。将行为封装在一个个独立的Strategy类中消除了这些条件语句。

例如，不用Strategy，正文换行的代码可能是像下面这样：
{% highlight c++ %}
void Composition::Repair(){
    switch(_breakingStrategy){
    case SimpleStrategy:
        CompseWithSimpleCompositor();
        break;
    case TeXStrategy:
        CompseWithTeXCompositor();
        break;
    // ...
    }
    // merge results with existing composition, 
    // if necessary.
}
{% endhighlight %}

Strategy模式将换行的任务委托给一个Strategy对象从而消除了这些case语句：
{% highlight c++ %}
void Composition::Repair(){
    _compositor->Compose();
    // merge results with existing composition, 
    // if necessary.
}
{% endhighlight %}

* `实现的选择`  Strategy模式可以提供相同行为的不同实现。客户可以根据不同时间/空间权衡取舍要求不同策略中选择进行。

* `客户必须了解不同的Strategy`  本模式有一个潜在的`缺点`，就是一个客户要选择一个合适的Strategy就必须知道这些Strategy到底有何不同。此时可能不得不向客户暴露具体的实现问题。因此仅当这些不同行为变体与客户相关的行为时，才需要使用Strategy模式。

* `Strategy和Context之间的通信开销`  无论各个ConcreteStrategy实现的算法时简单还是复杂，它们都共享Strategy定义的接口。因此很可能某些ConcreteStrategy不会都用到所有通过这个接口传递给他们的信息；简单的ConcreteStrategy可能不使用其中的任何信息！这就意味着有时Context会创建和初始化一些永远不会用到的参数。如果存在这样问题，那么将需要在Strategy和Context之间更进行紧密的耦合。

* `增加了对象的数目`  Strategy增加了一个应用中的对象数目。有时你可以将Strategy实现为可供各Context共享的无状态的对象来减少这一开销。任何其余的状态都由Context维护。Context在每一次对Strategy对象的请求中都将这个状态传递过去。共享的Strategy不应再各次调用之间维护状态。Flyweight模式更详细地描述了这一方法。

#### 9. 实现
* 定义Strategy和Context接口  Strategy和Context接口必须使得ConcreteStrategy能够有效的访问它所需要的Context中的任何数据，反之亦然。一种办法是让Context将数据放在参数中传递给Strategy操作 - 也就是说，将数据发送给Strategy。这使得Strategy和Context解耦。但另一方面，Context可能发送一些Strategy不需要的数据。

  另一种办法是让Context将自身作为一个参数传递给Strategy，该Strategy再显式地想该Context请求数据。或者，Strategy可以存储对它的Context的一个引用，这样根本不再需要传递任何东西。这两种情况下，Strategy都可以请求到它所需要的数据。但现在Context必须对它的数据定义一个更为精细的接口，这将Strategy和Context更紧密地耦合在一起。

* 将Strategy作为模板参数  在C++中，可利用模板机制用一个Strategy来配置一个类。
* 使Strategy对象成为可选的  如果即使在不使用额外的Strategy对象的情况下，Context也还有意义的话，那么它还可以被简化。Context在访问某Strategy前先检查它是否存在，如果有，那么就使用它；如果没有，那么Context执行缺省的行为。这种方法的好处是客户根本不需要处理Strategy对象，除非它们不喜欢缺省的行为。

#### 10. 代码示例

#### 11. 已知应用
* 编译器代码优化，Strategy定义了不同的寄存器分配方案和指令集调度策略。这就为在统统目标机器结构上实现优化程序提供了所需的灵活性。
* 计算引擎框架为不同的金融设备计算价格。
* 集成电路布局系统。
* ObjectWindows使用Validator对象来封装验证策略。

#### 12. 相关模式


























