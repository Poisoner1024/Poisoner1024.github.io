---
layout: post
title: "设计模式 - 行为模式 - 观察者"
description: "当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新"
tags: [design,pattern]
---

### 对象行为型模式 - 观察者（OBSERVER）

#### 1. 意图
定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。

Define a one-to-many dependency between objects so that when one object changes
state, all its dependents are notified and updated automatically.

#### 2. 别名
依赖（Dependents)，发布-订阅（Publish-Subscribe）

#### 3. 动机
将一个系统分割成一系列相互协作的类有一个常见的副作用：需要维护相关对象间的一致性。我们不希望为了维护一致性而使各类紧密耦合，因为这样降低了它们的可重用性。

例如，许多图形用户界面工具箱将用户应用的界面表示与地下的应用数据分离。定义应用数据的类和负责界面表示的类可以各自独立地复用。当然它们也可一起工作。一个表格对象和一个柱状图对象可使用不同的表示形式描述同一个应用数据对象的信息。表格对象和柱状图对象互相并不知道对方的存在，这样使你可以根据需要单独复用表格或柱状图。但是这里是它们**表现的**似乎互相知道。当用户改变表格中的信息时，柱状图能立即反映这一变化，反过来也是如此。

{% include image.html path="documentation/design-pattern/Observer-Demo.png" path-detail="documentation/design-pattern/Observer-Demo.png" alt="Observer-Demo" %}

这一行为意味着表格对象和柱状图对象都依赖于数据对象，因此数据对象的任何状态变化都应立即通知它们。同时也没有理由将依赖于该数据对象的对象的数目限定为两个，对相同的数据可以有任意数目的不同用户界面。

Observer模式描述了如何建立这种关系。这一模式中的关键对象是**目标（subject）**和**观察者（observer）**。一个目标可以有任意数目的依赖它的观察者。一旦目标的状态发生改变，所有的观察者都得到通知。作为对这个通知的响应，每个观察者都将查询目标以使其状态与目标的状态同步。

这种交互也称为**发布 - 订阅（publish-subscribe）**。目标是通知的发布者。它发出通知时并不需知道谁是它的观察者。可以有任意数目的观察者订阅并接收通知。

#### 4. 适用性
在以下任一情况下可以使用观察者模式：
* 当一个抽象模型有两个方面，其中一个方面依赖于另一方面。将这二者封装在独立的对象中以使它们可以各自独立地改变和复用。

  When an abstraction has two aspects, one dependent on the
  other. Encapsulating these aspects in separate objects lets you vary
  and reuse them independently.

* 当对一个对象的改变需要同时改变其它对象，而不知道具体有多少对象有待改变。
  
  When a change to one object requires changing others, and you don't know
  how many objects need to be changed.

* 当一个对象必须通知其它对象，而它又不能假定其它对象是谁。换言之，你不希望这些对象是紧密耦合的。
  
  When an object should be able to notify other objects without
  making assumptions about who these objects are. In other words, you don't want
  these objects tightly coupled.

#### 5. 结构
{% include image.html path="documentation/design-pattern/Observer-Pattern-Structure.png" path-detail="documentation/design-pattern/Observer-Pattern-Structure.png" alt="Observer-Pattern-Structure" %}

#### 6. 参与者
* Subject（目标）
    - 目标知道它的观察者。可以有任意多个观察者观察同一个目标。
    - 提供注册和删除观察者对象的接口。
* Observer（观察者）
    - 为那些在目标发生改变时需获得通知的对象定义一个更新接口。
* ConcreteSubject（具体目标）
    - 将有关状态存入各ConcreteObserver对象。
    - 当它的状态发生改变时，向它的各个观察者发出通知。
* ConcreteObserver （具体观察者）
    - 维护一个指向ConcreteSubject对象的引用。
    - 存储有关状态，这些状态与目标的状态保持一致。
    - 实现Observer的更新接口以使自身状态与目标的状态保持一致。

#### 7. 协作
* 当ConcreteSubject发生任何可能导致其观察者与其本身状态不一致的改变时，它将通知它的各个观察者。
* 在得到一个具体目标的改变通知后，ConcreteObserver对象可想目标对象查询信息。ConcreteObserver使用这些信息以使它的状态与目标对象的状态一致。
下面的交互图说明了目标对象和观察者之间的协作：
{% include image.html path="documentation/design-pattern/Observer-Pattern-Sequence-Diagram.png" path-detail="documentation/design-pattern/Observer-Pattern-Sequence-Diagram.png" alt="Observer-Pattern-Sequence-Diagram" %}
`注意`发出改变请求的Observer对象并不立即更新，而是将其推迟到它重目标得到一个通知之后。Notify不总是由目标对象调用。它也可被一个观察者或其它对象调用。在实现一节中将讨论一些常用的变化。

#### 8. 效果
Observer模式允许你独立的改变目标和观察者。你可以单独复用目标对象而无需同事复用其观察者，反之亦然。它也使你可以在不改动目标和其他的观察者的前提下增加观察者。
下面是观察者模式其他一些优缺点：
* `目标和观察者间的抽象耦合`

  一个目标所知道的仅仅是它有一系列观察者，每个都符合抽象的Observer类的简单接口。目标不知道任何一个观察者属于哪一个具体的类。这样目标和观察者之间的耦合是抽象的和最小的。

  因为目标和观察者不是紧密耦合的，它们可以属于一个系统中的不同抽象层次。一个处于较低层次的目标对象可以与一个处于较高层次的观察者通信并通知它，这样就保持了系统层次的完整。如果目标和观察者混在一起，那么得到的对象要么横贯两个层次（违反了层次性），要么必须放在这两层的某一层中（这可能会损害层次抽象）。

* `支持广播通信`
  
  不像通常的请求，目标发送的通知不需指定它的接收者。通知被自动广播给所有已向该目标对象登记的有关对象。目标对象并不关心到底有多少对象对自己感兴趣；它唯一的责任就是通知它的观察者。这给了你在任何时刻增加和删除观察者的自由。处理还是忽略一个通知取决于观察者。

* `意外的更新`

  因为一个观察者并不知道其他观察者的存在，它可能对改变目标的最终代价一无所知。在目标上一个看似无害的操作可能会引起一系列对观察者以及依赖于这些观察者的那些对象的更新。此外，如果依赖准则的定义或维护不当，常常会引起错误的更新，这种错误通常很难捕捉。

  简单的更新协议不提供具体说明目标中什么被改变了，这就使得上述问题更加严重。如果没有其他协议帮助观察者发现什么发生了改变，它们可能会被迫尽力减少改变。

#### 9. 实现
* `创建目标到其观察者之间的映射`  一个目标对象跟踪它应通知的观察者的最简单的方法是显式的在目标中保存对它们的引用。然而，当目标很多而观察者较少时，这样存储可能代价太高。一个解决办法就是用时间换空间，用一个关联查找机制（例如一个hash表）来维护目标到观察者的映射。这样一个没有观察者的目标就不产生存储开销。但另一方面，这一方法增加了访问观察者的开销。

* `观察多个目标`  在某些情况下，一个观察者依赖于多个目标可能是有意义的。例如，一个表格对象可能依赖于多个数据源。在这种情况下，必须扩展Update接口以使观察者知道是哪一个目标送来的通知。目标对象可以简单地将自己作为Update操作的一个参数，让观察者知道应去检查哪一个目标。

* `谁触发了更新`  目标和它的观察者依赖于通知机制来保持一致。但到底哪一个对象调用Notify来触发更新？此时有两个选择：
    - 由目标对象的状态设定操作在改变目标对象的状态后自动调用Notify。这种方法的优点是客户不需要记住要在目标对象上调用Notify，缺点是多个连续的操作会产生多次连续的更新，可能效率较低。
    - 让客户负责在适当的时候调用Notify。这样做的优点是客户可以再一系列的状态改变完成后再一次性地触发更新，避免了不必要的中间更新。缺点是给客户增加了触发更新的责任。由于客户可能会忘掉调用Notify，这种方式较易出错。

* `对已删除的目标的悬挂引用`  删除一个目标时应注意不要在其观察者中遗留对该目标的悬挂引用。一种避免悬挂引用的方法是，当一个目标被删除时，让它通知它的观察者将对该目标的引用复位。一般来说，不能简单地删除观察者，因为其他的对象可能会引用它们，或者也可能它们还在观察其他目标。

* `在发出通知前确保目标的状态自身是一致的`  在发出通知前确保目标的状态自身是一致这一点很重要，因为观察者在更新其状态的过程中需要查询目标的当前状态。
  当Subject的子类调用继承的该项操作时，很容易无意中违反这条自身一致的准则。例如，在下面的代码序列中，在目标尚处于一种不一致的状态时，通知就被触发了。
{% highlight c++ %}
    void MySubject::Operation(int newValue){
        BaseClassSubject::Operation(newValue);
          // trigger notification

        _myInstVar += newValue;
          // update subclass state (too late)
    }
{% endhighlight %}
  你可以用抽象的Subject类中的模板方法（Template Method）发送通知来避免这种错误。定义那些子类可以重定义的原语操作，并将Notify作为模板方法中的最后一个操作，这样当子类重定义了Subject的操作时，还可以保证该对象的状态是自身一致的。
{% highlight c++ %}
    void Text::Cut(TextRange r){
        ReplaceRange(r);    // redefined in subclasses
        Notify();
    }
{% endhighlight %}
顺便提一句，在文档中记录是哪一个Subject操作触发通知总是应该的。

* `避免特定于观察者的跟新协议 -- 推/拉模型`  观察者模式的实现经常需要让目标广播关于其改变的其他一些信息。目标将这些信息作为Update操作一个参数传递出去。这些信息的量可能很小，也可能很大。
  一个极端的情况是，目标向观察者发送关于改变的详细信息，而不管它们需要与否。我们称之为**推模型（push model）**。另一个极端是**拉模型（pull model）**；目标除最小通知外什么也不送出，而在此之后由观察者显式地向目标询问细节。

  拉模型强调的是目标不知道它的观察者，而推模型假定目标知道一些观察者的需要的信息。推模型可能使得观察者相对难以复用，因为目标对观察者的假定可能并不总是正确的。另一方面，拉模型可能效率较差，因为观察者对象需要在没有目标对象帮助下确定什么改变了。
  
* `显示地指定感兴趣的改变`  你可以扩展目标的注册接口，让各观察者注册为仅对特定事件感兴趣，以提高更新的效率。当一个事件发生时，目标仅通知那些已注册为对该时间感兴趣的观察者。支持这种做法一种途径是，对使用目标的**方面（aspects）**的概念。可用如下代码将观察者对象注册为对目标对象的某特定事件感兴趣：
ct的操作时，还可以保证该对象的状态是自身一致的。
{% highlight c++ %}
    void Subject::Attach(Observer*, Aspect& interest);
{% endhighlight %}
此处interest指定感兴趣的时间。在通知的时刻，目标将这方面的改变作为Update操作的一个参数提供给它的观察者，例如：
{% highlight c++ %}
    void Observer::Update(Subject*, Aspect& interest);
{% endhighlight %}

* `封装复杂的更新语义`  当目标和观察者间的依赖关系特别复杂时，可能需要一个维护这些关系的对象。我们称这样的对象为**更改管理器（ChangeManager）**。它的目的是尽量减少观察者反映其目标的状态变化所需要的工作量。例如，如果一个操作涉及到对几个相互依赖的目标进行改动，就必须保证仅在*所有的*目标都已更改完毕后，才一次性地通知它们观察者，而不是每个目标都通知观察者。

  ChangeManager有三个责任：
  - 它将一个目标映射到它的观察者并提供一个接口来维护这个映射。这就不需要由目标来维护对其观察者的引用，反之亦然。
  - 它定义一个特定的更新策略。
  - 根据一个目标的请求，它更新所有依赖于这个目标的观察者。
{% include image.html path="documentation/design-pattern/Observer-Pattern-Change-Manager.png" path-detail="documentation/design-pattern/Observer-Pattern-Change-Manager.png" alt="Observer-Pattern-Change-Manager" %}
  上图描述了一个简单的基于ChangeManager的Observer模式的实现。有两种特殊的ChangeManager。
  SimpleChangeManager总是更新每一个目标的所有观察者，比较简单。DAGChangeManager处理目标及其观察者之间依赖关系构成的无环有向图。当一个观察者观察多个目标时，DAGChangeManager要比SimpleChangeManager更好一些。在这种情况下，两个或更多目标中产生的改变可能会产生冗余更新。DAGChangeManager保证观察者仅接收一个更新。当然，当不存在多重更新的问题时，SimpleChangeManager更好一些。

  ChangeManager是一个Mediator模式的实例。通常只有一个ChangeManager，并且它是全局可见的。这个Singleton模式可能有用。

* `结合目标类和观察者类`  用不支持多重继承的语言（如SmallTalk）书写的类库通常不单独定义Subject和Observer，而是将它们的接口结合到一个类中。这就允许你定义一个既是一个目标又是一个观察者的对象，而不要要多重继承。例如在SmallTalk中，Subject和Observer接口定义于根类Object中，使得它们对所有的类都可用。

#### 10. 代码示例


#### 11. 已知应用
最著名的Observer模式的例子：Model/View/Controller。MVC的Model类担任目标的角色，而View是观察者的基类。

#### 12. 相关模式
* Mediator：通过封装复杂的更新语义，ChangeManager充当目标和观察者之间的中介者。
* Singleton： ChangeManager可使用Singleton模式来保证它是唯一的并且是可全局访问的。





















