---
layout: post
title: "什么是设计模式"
description: "可复用面向对象软件的基础"
tags: [design, pattern]
---
设计面向对象软件比较困难，而设计*可复用*的面向对象软件就更加困难。你必须找到相关的对象，以适当的颗粒度将他们归类，再定义类的接口和继承层次，建立对象之间的基本关系。你的设计应该对手头的问题有针对性，同时对将来的问题和需求也要有足够的通用性。你也希望避免重复设计或尽可能少做重复设计。这时候*设计模式*就登场了！我们做软件开发的过程中也不断的听到、提到*设计模式*，那究竟什么是*设计模式*呢？

Christopher Alexander说过：
> 每个模式模式描述了一个在我们周围不断重复发生的问题，以及该问题的解决方案的核心。这样，你就能一次又一次地使用该方案而不必做重复劳动

Christopher Alexander says, "Each pattern describes a problem which occurs over
and over again in our environment, and then describes the core of the solution
to that problem, in such a way that you can use this solution a million times
over, without ever doing it the same way twice".

尽管Alex所指的是城市和建筑模式，但他的思想也同样适用于面向对象设计模式，只是在面向对象的解决方案里，我们用对象和接口代替了墙壁和门窗。两类模式的核心都在于提供了相关问题的解决方案。

一般而言，一个模式有四个基本要素：
1. `模式名称`(pattern name) 一个助记名，它用一两个词来描述模式的问题、解决方案和效果。命名一个新的模式增加了我们的设计词汇。设计模式允许我们再较高的抽象层次上进行设计。基于一个模式的词汇表，我们自己以及同事之间就可以讨论模式并在编写文档时使用它们。模式名可以帮助我们思考，便于我们与其他人交流设计思想及设计结果。

2. `问题`(problem) 描述了应该在合适使用模式。它解释了设计问题和问题存在的前因后果，它能够描述了特定的设计问题，如怎样对对象表示算法等。也可能描述了导致不灵活设计的类或对象结构。有时候，问题部分会包括使用模式必须满足的一系列先决条件。

3. `解决方案`(solution) 描述了设计的组成部分，它们之间的相互关系及各自的职责和协作方式。因为模式就像一个模板，可应用于多种不同场合，所以解决方案并不描述一个特定而具体的设计或实现，而是提供设计问题的抽象描述和怎样用一个具有一般意思的元素组合（类或对象组合）来解决这个问题。

4. `效果`(consequences) 描述了模式应用效果及使用模式应权衡的问题。尽管我们描述设计决策时，并不总是提到模式效果，但对于评价设计选择和理解使用模式的代价和好处具有重要意义。软件效果大多关注对时间和空间的衡量，它们也表述了语言和实现问题。因为复用是面向对象设计的要素之一，所以模式效果包括对系统的灵活性、扩充性或可移植性的影响，显式地列出这些效果对理解和评价这些模式很有帮助。

一个设计模式命名、抽象确定了一个通用设计结构的主要方面，这些设计结构能被用来构造可复用的面
向对象设计。设计模式确定了所包含的类和实例，它们的角色、协作方式以及职责分配。每一个设计模式都集中于*一个特定的面向对象设计问题或设计要点，描述了什么时候使用它，在另一些设计约束条件下是否还能使用，以及使用的效果和如何取舍。*

A design pattern names, abstracts, and identifies the key aspects of a common
design structure that make it useful for creating a reusable object-oriented design.The design pattern identifies the participating classes and instances, their roles and collaborations, and the distribution of responsibilities. Each design pattern focuses on a particular object-oriented design problem or issue.It describes when it applies, whether it can be applied in view of other design constraints, and the consequences and trade-offs of its use.

### 设计模式分类

设计模式在粒度和抽象层次上各不相同。我们可以根据两条准则对模式进行分类。第一是**目的**准则，即模式是用来完成什么工作的。模式依据其目的可以分为**创建型（Creational）、结构型（Structural)、行为型（Bahavioral)**三种。创建型模式与对象的创建有关；结构型模式处理类或者对象的组合；行为型模式对类或对象怎样交互和怎样分配职责进行描述。第二是**范围**准则，指定模式主要是用于类还是用于对象。类模式处理类和子类之间的关系，这些关系通过继承建立，是静态的，在编译时刻便确定下来了。对象模式处理对象间的关系，这些关系在运行时刻是可以变化的，更具动态性。从某种意义上来说，几乎所有模式都使用继承机制，所以“类模式”只指那些集中于处理类间关系的模式，而大部分模式都属于对象模式的范畴。

创建型类模式将对象的部分创建工作延迟到子类，而创建型对象模式则将它延迟到另一个对象中。结构型类模式使用继承机制来组合类，而结构型对象模式则描述了对象的组装方式。行为型类模式使用继承描述算法和控制流，而行为型对象模式则描述一组对象怎样协作完成单个对象所无法完成的任务。

Design patterns vary in their granularity and level of abstraction. We classify design patterns by two criteria. The first criterion, called purpose, reflects what a pattern does. Patterns can have either creational, structural, or behavioral purpose. Creational patterns concern the process of object creation.Structural patterns deal with the composition of classes or
objects. Behavioral patterns characterize the ways in which classes or objects
interact and distribute responsibility. The second criterion, called scope, specifies whether the pattern applies primarily to classes or to objects. Class patterns deal with relationships between classes and their subclasses. These relationships are established through inheritance, so they are static—fixed at compile-time. Object patterns deal with object relationships, which can be changed at run-time and are more dynamic. Almost all patterns use inheritance to some extent. So the only patterns labeled "class patterns" are those that focus on class relationships. Note that most patterns are in the Object scope.

Creational class patterns defer some part of object creation to subclasses, while Creational object patterns defer it to another object. The Structural class patterns use inheritance to compose classes, while the Structural object patterns describe ways to assemble objects. The Behavioral class patterns use inheritance to describe algorithms and flow of control, whereas the Behavioral object patterns describe how a group of objects cooperate to perform a task that no single object can carry out alone.





