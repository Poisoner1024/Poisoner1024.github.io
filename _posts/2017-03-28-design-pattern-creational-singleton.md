---
layout: post
title: "设计模式 - 创建型模式 - 单件"
description: "保证一个类仅有一个实例"
tags: [design,pattern]
---

### 对象创建型模式 -- 单件（SINGLETON）

#### 1. 意图
保证一个类仅有一个实例，并提供一个访问它的全局访问点。

#### 2. 动机
对一些类来说，只有一个实例是很重要的。虽然系统中可以有许多打印机，但却只应该由一个打印机脱机（printer spooler)，只应该有一个文件系统和一个窗口管理器。一个数字滤波器只能有一个A/D转换器。一个会计系统只能专用于一个公司。

我们怎么样才能保证一个类只有一个实例并且这个实例易于被访问呢？一个全局变量使得一个对象可以被访问，但它不能防止你实例化多个对象。

一个更好的办法是，让类自身负责保存它的唯一实例。这个类可以保证没有其他实例可以被创建（通过截取创建新对象的请求），并且它可以提供一个访问该实例的方法。这就是Singleton模式。

#### 3. 适用性
* 当类只能有一个实例而且客户可以从一个众所周知的访问点访问它时。
* 当这个唯一的实例应该是通过子类化可扩展的，并且客户应该无需更改代码就能使用一个扩展的实例时。

#### 4. 结构
{% include image.html path="documentation/design-pattern/Singleton-Pattern-Structure.png" path-detail="documentation/design-pattern/Singleton-Pattern-Structure.png" alt="Singleton-Pattern-Structure" %}

#### 5. 参与者
* Singleton
    - 定义一个Instance操作，允许客户访问它的唯一实例。Instance是一个类操作（即Smalltalk中的一个类方法和C++中的一个静态成员函数）。
    - 可能复杂创建它自己的唯一实例。

#### 6. 协作
客户只能通过Singleton的Instance操作访问一个Singleton的实例。

#### 7. 效果
Singleton模式有许多有点：
* 1）`对唯一实例的受控访问`  因为Singleton类封装它的唯一实例，所以它可以严格的控制客户怎样以及何时访问它。
* 2）`缩小名空间`  Singleton模式是对全局变量的一种改进。它避免了哪些存储唯一实例的全局变量污染名空间。
* 3）`允许对操作和表示的精化`  Singleton类可以有子类，而且用这个扩展类的实例配置一个应用是很容易的。你可以用你所需要的类的实例在运行时刻配置应用。
* 4）`允许可变数目的实例`  这个模式使得你易于改变你的想法，并允许Singleton类的多个实例。此外，你可以用相同的方法来控制应用所使用的实例的数目。只有允许访问Singleton实例的操作需要改变。
* 5）`比类操作更灵活`  另一种封装单件功能的方式是使用类操作（即C++中的静态成员函数或者是Smalltalk中的类方法）。但这两种语言技术都难以改变设计以允许一个类有多个实例。此外，C++中的静态成员函数不是虚函数，因此子类不能多态的重定义它们。

#### 8. 实现
* 1）保证一个唯一的实例  Singleton模式使得这个唯一实例是类的一般实例，但该类被写成只有一个实例能被创建。做到这一点的一个常用方法是将创建这个实例的操作隐藏在一个类操作（即一个静态成员函数或者是一个类方法）后面，由它保证只有一个实例被创建。这个操作可以访问保存唯一实例的变量，而且它可以保证这个变量在返回值之前用这个唯一实例初始化。这种方法保证了单件在它的首次使用前被创建和使用。
* 2）创建Singleton类的子类  主要问题与其说是定义子类不如说是建立它的唯一实例，这样客户就可以使用它。事实上，指向单件实例的变量必须用子类的实例进行初始化。最简单的技术是在Singleton的Instance操作中决定你想使用的是哪一个单件。代码示例一节中的一个例子说明了如何用环境变量实现这一技术。
  
  另一个选择Singleton的子类的方法是将Instance的实现从父类中分离出来并将它放入子类。这就允许C++程序员在链接时刻决定单件的类（即通过链入一个包含不同实现的对象文件），但对单件的客户则隐藏这一点。

  链接的方法在链接时刻确定了单件类的选择，这使得难以在运行时刻选择单件类。使用条件语句来决定子类更加灵活一些，但这硬性限定（hard-wire）了可能的Singleton类的集合。这两种方法不是在所有的情况都足够灵活。

  一个更灵活的方法是使用一个**单件注册表（registry of singleton）** 。可能的Singleton类的集合不是由Instance定义的，Singleton类可以根据名字在一个众所周知的注册表中注册它们的单件实例。这个注册表在字符串名字和单件之间建立映射。当Instance需要一个单件时，它参考注册表，根据名字请求单件。

  注册表查询相应的单件（如果存在的话）并返回它。这个方法使得Instance不再需要知道所有可能的Singleton类或实例。它所需要的只是所有Singleton类的一个公共的接口，该接口包括了对注册表的操作：
{% highlight c++ %}
class Singleton{
public:
    static void Register(const char* name, Singleton*);
    static Singleton* instance();
protected:
    static Singleton* Lookup(const char* name);
private:
    static Singleton* _instance;
    static List<NameSingletonPair>* _registry;
}
{% endhighlight %}
Register以给定的名字注册Singleton实例。为保证注册表简单，我们将它存储一列NameSingletonPair对象。每个NameSingletonPair将一个名字映射到一个单件。Lookup操作根据给定单件的名字进行查找。我们假定一个环境变量指定了所需要的单件的名字。
{% highlight c++ %}
Singleton* Singleton::Instance(){
    if(_instance == 0){
        const char* singletonName =  getenv("SINGLETON");
        // user or environment supplies this at startup

        _instance = Lookup(singletonName);
        // Lookup return 0 if there's no such singleton
    }
    return _instance;
}
{% endhighlight %}
Singleton类在何处注册它们自己？一种可能是在它们的构造器中。例如，MySingleton子类可以像下面这样做：
{% highlight c++ %}
MySingleton::MySingleton(){
    // ...
    Singleton::Register("MySingleton", this);
}
{% endhighlight %}
当然，除非实例化类否则这个构造器不会被调用，这正反映了Singleton模式试图解决的问题！在C++中我们可以定义MySingleton的一个静态实例来避免这个问题。例如我们可以在包含MySingleton实现的文件中定义：
{% highlight c++ %}
static MySingleton thisSingleton;
{% endhighlight %}
Singleton类不再负责创建单件。它的主要职责是使用供选择的单件对象在系统中可以被访问。静态对象方法还是有一个潜在的缺点 -- 也就是所有可能的Singleton子类的实例都必须被创建，否则它们不会被注册。

#### 9. 代码示例

#### 10. 已知应用

#### 11. 相关模式
很多模式可以使用Singleton模式实现。如，Abstract Factory，Builder和Prototype。

















