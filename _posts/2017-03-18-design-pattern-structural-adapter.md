---
layout: post
title: "设计模式 - 结构型模式 - 适配器"
description: "将一个类的接口转换成客户希望的另外一个接口"
tags: [design,pattern]
---

### 类对象结构型模式 - 适配器（ADAPTER）


#### 1. 意图
将一个类的接口转换成客户希望的另外一个接口。Adapter模式使得原本由于接口不兼容而不能在一起工作的那些类可以一起工作。

#### 2. 别名
包装器Wrapper。

#### 3. 动机
有时，为复用而设计的工具箱不能够被复用的原因仅仅是因为它的接口与专业应用领域所需要的接口不匹配。

例如，有一个绘图编辑器，这个编辑器允许用户绘制和排列基本图元（线、多边型和正文等）生成图片和图表。这个绘图编辑器的关键抽象是图形对象。图形对象有一个可编辑的形状，并可以绘制自身。图形对象的接口由一个称为Shape的抽象类定义。绘图编辑器为每一种图形对象定义了一个Shape的子类：LineShape对应于直线，PolygonShape类对应于多变型，等等。

像LineShape和PolygonShape这样的基本几何图形的类比较容易实现，这是由于他们的绘图和编辑功能本来就很有限。但是对于可以显示和编辑正文的TextShape子类来说，实现相当困难，因为即使是基本的正文编辑也要涉及到复杂的屏幕刷新和缓存区管理。同时成品的图形用户界面工具箱可能已经提供了一个复杂的TextView类用于显示和编辑正文。理想的情况下，我们可以复用这个TextView类以实现TextShape类，但是工具箱的设计者当时并没有考虑Shape的存在，因此TextView和Shape对象不能互换。

一个应用可能会有一些具有不同的接口并且这些接口互不兼容，在这样的应用中像TextView这样已经存在并且不相关的类如何协同工作呢？我们可以改变TextView类使它兼容Shape类接口，但前提是必须有这个工具箱的源代码。然而即使我们得到了这些源代码，修改TextView也是没有什么意义的；因为不应该仅仅为了实现一个应用，工具箱就不得不采用一些与特定领域相关的接口。

我们可以不用上门的方法，而定义一个TextShape类，由它来*适配*TextView的接口和Shape的接口。我们可以用两种方法做这件事情：1）继承Shape类的接口和TextView的实现，或2）将一个TextView实例作为TextShape的组成部分，并且使用TextView的接口实现TextShape，这两种方法恰恰对应于Adapter模式的类和对象版本。我们将TextShape称之为**适配器Adapter**。
{% include image.html path="documentation/design-pattern/Demo-Adapter.png" path-detail="documentation/design-pattern/Demo-Adapter.png" alt="Demo-Adapter" %}
上面的类图说明了对象适配器实例。它说明了在Shape类中声明的BoundingBox请求如何被转换成在TextView类中定义的GetExtent请求。由于TextShape将TextView的接口与Shape的接口进行了匹配，因此绘图编辑器就可以服用原先并不兼容的TextView类。

Adapter时常还要负责提供哪些被匹配的类所没有提供的功能，上面的类图中说明了适配器如何实现这些职责。由于绘图编辑器允许用户交互的将每一个Shape对象“拖动”到一个新的位置，而TextView设计中没有这中功能。我们可以实现TextShape类的CreateManipulator操作，从而增加这个缺少的功能，这个操作返回相应的Manipulator子类的一个实例。

Manipulator是一个抽象类，它所描述的对象知道如何驱动Shape类响应相应的用户输入，例如将图形拖动到一个新的位置。对应于不同形状的图形，Manipulator有不同的子类；例如子类TextManipulator对应TextShape。TextShape通过返回一个TextManipulator实例，增加了TextView中缺少而Shape需要的功能。

#### 4. 适用性
以下情况使用Adapter模式：
* 你想使用一个已经存在的类，而它的接口不符合你的需求。
* 你想创建一个可以复用的类，该类可以与其他不相关的类或不可以预见的类（即那些接口可能不一定兼容的类）协同工作。
* （仅适用于对象Adapter）你想使用一些已经存在的子类，但是不可能对每一个都进行子类化以匹配它们的接口。对象适配器可以适配它的父类接口。

#### 5. 结构
类适配器使用多重继承对一个接口与另一个接口进行匹配，对象匹配器依赖于对象组合。
{% include image.html path="documentation/design-pattern/Adapter-Pattern-Structure.png" path-detail="documentation/design-pattern/Adapter-Pattern-Structure.png" alt="Adapter-Pattern-Structure" %}

#### 6. 参与者
* Target（Shape）
    - 定义Client使用的与特定领域相关的接口。
* Client（DrawingEditor)
    - 与符合Target接口的对象协作
* Adaptee（TextView）
    - 定义一个已经存在的接口，这个接口需要适配。
* Adapter（TextShape）
    - 对Adaptee的接口与Target接口进行适配

#### 7. 协作
Client在Adapter实例上调用一些操作。接着适配器调用Adaptee的操作实现这个请求。

#### 8. 效果
类适配器和对象适配器有不同的权衡。类适配器：
* 用一个具体的Adapter类对Adapter和Target进行匹配。结果是当我们想要匹配一个类以及所有它的子类时，类Adapter将不能胜任工作。
* 使得Adapter可以重定义Adaptee的部分行为，因为Adapter是Adaptee的一个子类。
* 仅仅引入了一个对象，并不需要额外的指针（或引用）以间接的得到adaptee。

对象适配器则：
* 允许一个Adapter与多个Adaptee--即Adaptee本身以及他的所有子类（如果有子类的话）--同事工作。Adapter也可以一起给所有的Adaptee添加功能。
* 使得重定义Adaptee的行为比较困难。这就需要生成Adaptee的子类并且使得Adapter引用这个子类而不是引用Adaptee本身。

使用Adapter模式时需要考虑的其他一些因素有：
* Adapter的匹配程度
  对Adaptee的接口与Target的接口进行匹配的工作量哥哥Adapter可能不一样。工作范围可能是，从简单的接口转换（例如改变操作名）到支持完全不同的操作集合。Adapter的工作量取决于Target接口与Adaptee接口的相似程度。
* 可插入的Adapter
  当其他的类使用一个类时，如果所需的假定条件越少，这个类就更具有复用性。如果将接口匹配构建为一个类时，就不需要假定对其他的类可见的是一个相同的接口。也就是说，接口匹配使得我们可以将自己的类加入到一些现有的系统中去，而这些系统对这个类的接口可能会有所不同。
* 使用双向适配器提供透明操作
  使用适配器的一个潜在问题是，它们不对有的客户都透明。被适配的对象不再兼容Adaptee的接口，因此并不是所有Adaptee对象可以被使用的地方它都可以使用。双向适配器提供了这样的透明性。这两个不同的客户需要用不同的方式查看同一个对象时，双向适配器尤其有用。
  
#### 9. 实现
尽管Adapter模式的实现方式通常简单直接，但是仍需要注意以下一些问题：
* 使用C++实现适配器类  有许多方法可以实现可插入的适配器。
    - 使用抽象操作
    - 使用代理对象
    - 参数化适配器

#### 10. 代码示例

#### 11. 已知应用

#### 12. 相关模式
模式Bridge的结构与对象适配器类似，但是Bridge模式的出发点不同：Bridge目的是将接口部分和实现部分分离，从而对它们可以较为容易也相对独立的加以改变。而Adapter则意味着改变一个已有对象的接口。

Decorator模式增强了其他对象的功能而同时又不改变它的接口。因此Decorator对应用程序的透明性比适配器要好。结果是Decorator支持递归组合，而纯粹使用适配器是不可能实现这一点的。

模式Proxy在不改变它的接口的条件下，为另一个对象定义了一个代理。


