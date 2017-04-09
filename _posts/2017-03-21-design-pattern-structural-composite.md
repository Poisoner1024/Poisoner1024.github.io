---
layout: post
title: "设计模式 - 结构型模式 - 组合"
description: "将对象组合成树形结构以表示‘部分-整体’的层次结构"
tags: [design,pattern]
---

### 对象结构型模式 - 组合（COMPOSITE）
{:.no_toc}

* 目录
{:toc}

#### 1. 意图
将对象组合成树形结构以表示“部分-整体”的层次结构。Composite使得用户对当个对象和组合对象的使用具有一致性。

#### 2. 动机
在绘图编辑器和图形捕捉系统这样的图形应用程序中，用户可以使用简单的组件创建复杂的图表。用户可以组合多个简单组件以形成一些较大的组件，这些组件又可以组合成更大的组件。一个简单的实现方法是Text和Line这样的图元定义一些类，另外定义一些类作为这些图元的容器类（Container）。

然后这种方法存在一个问题：使用这些类的代码必须区别对待图元对象与容器对象，而实际上大多数情况下用户认为它们是一样的。对这些类区别使用，使得程序更加负责。Composite模式描述了如何使用递归组合，使得用户不必对这些类进行区别，如下图所示。
{% include image.html path="documentation/design-pattern/Composite-Demo.png" path-detail="documentation/design-pattern/Composite-Demo.png" alt="Composite-Demo" %}
Composite模式的关键是一个抽象类，它既可以代表图元，又可以代表图元的容器。在图形系统中这个类就是Graphic，它声明一些与特定图形对象相关的操作，例如Draw。同时它也声明了所有的组合对象共享的一些操作，例如一些操作用于访问和管理它的子部件。

子类Line，Rectangle和Text定义了一些图元对象，这些类实现Draw，分别用于绘制直线、矩形和正文。由于图元都没有子图形，因此它们都不执行与子类有关的操作。

Picture类定义了一个Graphic对象的聚合。Picture的Draw操作是通过对它的子部件调用Draw实现的，Picture还用这种方法实现了一些与其子部件相关的操作。由于Picture接口与Graphic接口是一致的，因此Picture对象可以可以递归的组合对象结构。

#### 3. 适用性
* 你想要表示对象的部分--整体层次结构
* 你希望用户忽略组合对象与单个对象的不同，用户将图一地使用组合结构中的所有对象。

#### 4. 结构
{% include image.html path="documentation/design-pattern/Composite-Structure.png" path-detail="documentation/design-pattern/Composite-Structure.png" alt="Composite-Structure" %}

典型的Composite对象结构如下图所示：
{% include image.html path="documentation/design-pattern/Composite-Relationship.png" path-detail="documentation/design-pattern/Composite-Relationship.png" alt="Composite-Relationship" %}

#### 5. 参与者
* Component（Graphic）
    - 为组合中的对象声明接口。
    - 在适当的情况下，实现所有类共有接口的缺省行为。
    - 声明一个接口用于访问和管理Component的子组件
    - （可选）在递归结构中定义一个接口，用于访问一个父组件，并在合适的情况下实现它。
* Leaf（Rectangle，Leaf，Text等）
    - 在组合中表示叶节点对象，叶节点没有子节点
    - 在组合中定义图元对象的行为。
* Composite（Picture）
    - 定义有子部件的那些部件的行为。
    - 存储子部件。
    - 在Component接口中实现与子部件有关的操作。
* Client 
    - 通过Component接口操纵组合部件的对象。

#### 7. 效果
* 定义了包含基本对象和组合对象的类层次结构  基本对象可以被组合成更复杂的组合对象，而这个组合对象又可以被组合，这样不断的递归下去。客户代码中，任何用到基本对象的地方都可以使用组合对象。

* 简化客户代码  客户可以一致的使用组合结构和单个对象。通常用户不知道（也不关心）处理的是一个叶节点还是一个组合组件。这就简化了客户代码，因为在定义组合的那些类中不需要写一些充斥着选择语句的函数。

* 使得更容易增加新类型的组件  新定义的Composite或Leaf子类自动的与已有的结构和客户代码一起工作，客户程序不需要因新的Component类而改变。

* 使你的设计变得更加一般化  容易增加新组合也会产生一些问题，那就是很难限制组合中的组件。有时你希望一个组合只能有某些特定的组件。使用Composite时，你不能依赖类型系统施加这些约束，而必须在运行时刻进行检查。

#### 8. 实现
我们在实现Composite模式使需要考虑以下几个问题：
* `显式的父部件引用`  保存从子部件到父部件的引用能简化组合结构的遍历和管理。父部件引用可以简化结构的上移和组件的删除，同时父部件引用也支持Chain of Responsibility模式。

* `共享组件`  共享组件是很有用的，比如它可以减少对存储的需求。但是当一个组件只有一个父部件时，很难共享组件。

* `最大化Component接口`  Composite模式的目的之一是使得用户不知道他们正在使用的具体的Leaf和Composite类。为了达到这一目的，Composite类应为Leaf和Composite类尽可能多定义一些公共操作。Composite类通常为这些操作提供缺省的实现，而Leaf和Composite子类可以对它们进行重定义。

* `声明管理子部件的操作`  虽然Composite类实现了Add和Remove操作用于管理子部件，但是Composite模式中一个重要的问题是：在Composite类层次结构中哪一些类声明这些操作。我们是应该在Component中声明这些操作，并使这些操作对Leaf类有意义呢，还是只应该在Composite和它的子类中声明并定义这些操作呢？这需要在安全性和透明性之间做出权衡选择。

* `Component是否应该实现一个Component列表`  你可能希望在Component类中奖子节点集合定义为一个实例变量，而这个Component类中也声明了一些操作对子节点进行访问和管理。但是在基类中存放子类指针，对叶节点来说会导致空间浪费，因为叶节点根本没有子节点。只有当该结构中子类数目相对较少时，才值得使用这种方法。

* `子部件排序`  许多设计指定了Composite的子部件顺序。在前面的Graphic例子中，排序可能表示了从前至后的顺序。如果Composite表示语法分析树，Composite子部件的顺序必须反映程序结构，而组合语句就是这样一些Composite的实例。

* `使用高速缓冲存储改善性能`  如果你需要对组合进行频繁的遍历或查找。Composite类可以缓存存储对它的子节点进行遍历或查找的相关信息。Composite可以缓冲存储你实际结构或者仅仅是一些用于缩短遍历或查找长度信息。

* `应该由谁删除Component`  在没有垃圾回收机制的语言中，当一个Composite被销毁时，通常最好由Composite负责删除其子节点。但有一种情况除外，即Leaf对象不会改变，因以被共享。

* `存储组件最好用哪一种数据结构` Composite可以使用多种数据结构存储它们的子节点，包括连接列表、树、数组和hash表。数据结构的选择取决于效率。事实上，使用通用的数据结构根本没有必要。有时对每个子节点，Composite都有一个变量与之对应，这就要求Composite的每个子类都要实现自己的管理接口。可以参考Interpreter模式。

#### 9. 代码示例
计算机和立体声组合音响这样的设备经常被组装成`部分-整体层次结构`或者是`容器层次结构`。例如，底盘可包含驱动装置和平面板，总线含有多个插件，机柜包括底盘、总线等。这种结构可以很自然地用Composite模式进行模拟。

Equipment类为在部分 - 整体层次结构中的所有设备定义一个接口。
{% highlight java %}
public abstract class Equipment {
    private String _name;

    Equipment(){

    }

    public String name(){
        return _name;
    }

    abstract Watt power();
    abstract Currency netPrice();
    abstract Currency discountPrice();

    abstract void add(Equipment eq);
    abstract void remove(Equipment eq);
    abstract Iterator<Equipment> createIterator();

    Equipment(String name){

    }
}
{% endhighlight %}
Equipment的子类包括表示磁盘驱动器、集成电路和开关的Leaf类：
{% highlight java %}
public abstract class FloppyDisk extends Equipment {
    FloppyDisk(String name){
        this._name = name;
    }

    abstract Watt power();
    abstract Currency netPrice();
    abstract Currency discountPrice();

    private String _name;
}
{% endhighlight %}
CompositeEquipment是包含其他设备的基类，它也是Equipment的子类。
{% highlight java %}
public abstract class CompositeEquipment extends Equipment {
    CompositeEquipment(){

    }

    CompositeEquipment(String name){
        this._name = name;
    }

    abstract Watt power();
    abstract Currency netPrice();
    abstract Currency discountPrice();

    abstract void add(Equipment eq);
    abstract void remove(Equipment eq);
    abstract Iterator<Equipment> createIterator();


    private List<Equipment> _equipment;
    private String _name;
}
{% endhighlight %}
现在我们将计算机的底盘表示为CompositeEquipment的子类Chassis。Chassis从CompositeEquipment继承了与子类有关的那些操作。
{% highlight java %}
public abstract class Chassis extends CompositeEquipment {

    Chassis(String name){
        this._name = name;
    }

    abstract Watt power();
    abstract Currency netPrice();
    abstract Currency discountPrice();

    private String _name;
}
{% endhighlight %}
我们可用相似的方式定义其他设备容器，如Cabinet和Bus。这样我们就得到了组装一台（非常简单）个人计算机所需的所有设备
{% highlight java %}
public class Client {
    public void main(String args){
        Cabinet cabinet = new Cabinet("PC Cabinet");
        Chassis chassis = new Chassis("PC Chassis");
        
        cabinet.add(chassis);
        
        Bus bus = new Bus("MCA Bus");
        bus.add(new Card("16Mbs Token Ring"));

        chassis.add(bus);
        chassis.add(new FloppyDisk("3.5in Floppy"));
    }
}
{% endhighlight %}

#### 10. 已知应用
几乎在所有面向对象的系统中都有Composite模式的应用实例。

#### 11. 相关模式
通常部件 - 父部件连接用于Chain of Responsibility模式。

Decorator模式经常与Composite模式一起使用。当装饰与组合一起使用时，它们通常有一个公共的父类。因此装饰必须支持具有Add、Remove和GetChild操作的Component接口。

Flyweight让你共享组件，但不再能引用它们的父部件。

Iterator可以用来遍历Composite。

Visitor将本来应该分布在Composite和Leaf类中的操作和行为局部化。
























