---
layout: post
title: "设计模式 - 行为模式 - 开篇"
description: "算法和对象间职责的分配"
tags: [design,pattern]
---

### 行为模式

`行为模式涉及到算法和对象间职责的分配`。行为模式不仅描述对象或类的模式，还描述了它们之间的通信模式。这些模式刻画了在运行时难以跟踪的复杂的控制流。它们将你的注意力从控制流转移对象间的联系方式上来。

Behavioral patterns are concerned with algorithms and the assignment of
responsibilities between objects. Behavioral patterns describe not just patterns
of objects or classes but also the patterns of communication between them. These
patterns characterize complex control flow that's difficult to follow at run-time.
They shift your focus away from flow of control to let you concentrate just on
the way objects are interconnected.

行为类模式使用`继承机制`在类间分派性行为。其中Template Method较为简单和常用。模板方法是一个算法的抽象定义，它逐步地定义该算法，每一步调用一个抽象操作或一个原语操作，子类定义抽象操作以具体实现该算法。另一种行为类模式是 Intepreter。它将一个文法表示为一个类层次，并实现一个解释器作为这些类的实例上的一个操作。

行为对象模式使用`对象复合`而不是继承。一些行为对象模式描述了一组对等的对象怎样相互协作以完成其中任一个对象都无法单独完成的任务。这里一个重要的问题是对等的对象如何互相了解对方。对等对象可以保持显式的对对方的引用，但那会增加它们的耦合度。在极端情况下，每一个对象都要了解所有其他的对象。Mediator在对等对象间引入一个Mediator对象以避免这个情况的出现。mediator提供了松耦合所需的间接性。

Chain of Responsibility提供更松的耦合。它让你通过一条候选对象链隐式的向一个对象发送请求。根据运行时刻情况任一候选者都可以响应相应的请求。候选者的数目是任意的，你可以再运行时刻决定哪些候选者参与到链中。

Observer模式定义并保持对象间的依赖关系。典型的Observer的例子是Smalltalk中的模式/视图/控制器，其中一旦模型的状态发生变化，模型的所有视图都会得到通知。

其他的行为对象模式常将行为封装在一个对象中并将请求指派给它。Strategy模式将算法封装在对象中，这样可以方便地指定和改变一个对象所使用的算法。Command模式将请求封装在对象中，这样它可作为参数来传递，也可以被存储在历史列表里，或者以其他方式使用。State模式封装一个对象的状态，使得当这个对象的状态对象变化时，该对象可改变它的行为。Visitor封装分布于多个类之间的行为，而Iterator则抽象了访问和遍历一个集合中的对象的方式。