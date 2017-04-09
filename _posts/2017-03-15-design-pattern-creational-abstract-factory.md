---
layout: post
title: "设计模式 - 创建型模式 - 抽象工厂"
description: "提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类"
tags: [design,pattern]
---

### 对象创建型模式 -- 抽象工厂（Abstract Factory）
{:.no_toc}

* 目录
{:toc}

#### 1. 意图
提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。

#### 2. 别名
Kit

#### 3. 动机
考虑一个支持多种视感（look-and-feel）标准的用户界面工具包，例如Motif和Presentation Manager。不同的视感风格为诸如滚动条、窗口和按钮等用户界面“窗口组件”定义不同的外观和行为。为保证视感风格标准间的可移植性，一个应用不应该为一个特定的视感外观硬编码它的窗口组件。在整个应用中实例化特定视感风格的窗口组件类将使得以后很难改变视感风格。

为解决这一问题我们可以定义一个抽象的WidgetFactory类，这个类声明了一个用来创建每一类基本窗口组件的接口。对于每一个抽象窗口组件类，WidgetFactory接口都有一个返回新窗口组件对象的操作。客户调用这些操作以获得窗口组件实例，但客户并不知道他们正在使用的哪些具体类。这样客户就不依赖于一般的视感风格，如下图所示：
{% include image.html path="documentation/design-pattern/Demo-Abstract-Factory.png" path-detail="documentation/design-pattern/Demo-Abstract-Factory.png" alt="Demo-Abstract-Factory" %}
每一个视感标准都对应于一个具体的WidgetFactory子类。每一个子类实现那些用于创建合适视感风格的窗口组件的操作。例如，MotifWidgetFactory的CreateScrollBar操作实例化并返回一个Motif滚动条，而相应的PMWidgetFactory操作返回一个Presentation Manager的滚动条。客户仅与抽象类定义的接口交互，而不使用特定的具体类的接口。

WidgetFactory也增强了具体窗口组件类之间的依赖关系。一个Motif的滚动条应该与Motif的按钮、Motif正文编辑器一起使用，这一约束条件作为使用MotifWidgetFactory的结果被自动加上。

#### 4. 适用性
在以下情况可以使用Abstract Factory模式
* 一个系统要独立于它的产品的创建、组合和表示时。
* 一个系统要由多个产品系列中的一个来配置时。
* 当你要强调一系列相关的产品对象的设计以便进行联合使用时。
* 当你提供一个产品类库，而只想显示他们的接口而不是实现时。

#### 5. 结构
{% include image.html path="documentation/design-pattern/Abstract-Factory-Structure.png" path-detail="documentation/design-pattern/Abstract-Factory-Structure.png" alt="Abstract-Factory-Structure" %}

#### 6. 参与者
* AbstractFactory（WidgetFactory) 
    - 声明一个创建抽象产品对象的操作接口
* ConcreteFactory（MotifWidgetFactory，PMWidgetFactory）
    - 实现创建具体产品对象的操作
* AbstractProduct（Windows，ScrollBar）
    - 为一类产品对象声明一个接口
* ConcreteProduct（MotifWindow， MotifScrollBar）
    - 定义一个将被相应的具体工厂创建的产品对象
    - 实现AbstractProduct接口
* Client
    - 仅适用由AbstractFactory和AbstractProduct类声明接口

#### 7. 协作
* 通常在运行时刻创建一个ConcreteFactory类的实例。这一具体的工厂创建具有特定实现的产品对象。为创建不同的产品对象，客户应使用不同的具体工厂。
* AbstractFactory将产品对象的创建延迟到它的ConcreteFactory子类。

#### 8. 效果
AbstractFactory模式有下面的一些优点和缺点：
* 它分离了具体的类

  Abstract Factory模式帮助你控制一个应用创建的对象的类。因为一个工厂封装创建产品对象的责任和过程，它将客户与类的实现分离。客户通过它们的抽象接口操作实例。产品的类名也在具体工厂的实现中被分离；它们不出现在客户代码中。

* 它使得易于交互产品系列

  一个具体工厂类在一个应用中仅出现一次--即在它初始化的时候。这使得改变一个应用的具体的具体工厂变得很容易。它只需改变具体的工厂即可使用不同的产品配置，这是因为一个抽象工厂创建了一个完整的产品系列，所以整个产品系列会立刻改变。在我们的用户界面的例子中，我们仅需转换到对应的工厂对象并重新创建接口，就可以实现从Motif窗口组件转换为Presentation Manager窗口组件。
* 它有利于产品的一致性

  当一个系列中的产品对象呗设计成一起工作时，一个应用一次只能使用同一个系列中的对象，这一点很重要。而AbstractFactory很容易实现这一点。

* 难以支持新种类的产品

  难以扩展抽象工厂以生产新种类的产品。这是因为AbstractFactory接口确定了可以被创建的产品集合。支持新种类的产品就需要扩展该工厂接口，这将涉及AbstractFactory类及其所有子类的改变。**我们会在实现一节讨论这个问题的一个解决办法。**

#### 9. 实现
下面是实现Abstract Factory模式的一些有用技术：
* 将工厂作为单件
  
  一个应用中一般每个产品系列只需要一个ConcreteFactory的实例。因此工厂通常最好实现为一个Singleton。

* 创建产品
  
  AbstractFactory仅声明一个创建产品的*接口*，真正创建产品是由ConcreteProduct子类实现的。最通常的一个办法是为每一个产品定义一个工厂方法（参见Factory Method）。一个具体的工厂将为每个产品重定义该工厂方法以指定产品。虽然这样的实现很简单，但它却要求每个产品系类都要有一个新的具体工厂子类，即使这些产品系列的差别很小。

  如果有多个可能的产品系列，具体工厂也可以使用Prototype模式来实现。具体工厂使用产品系列中每一个产品的原型实例来初始化，且它通过复制它的原型来创建新的产品。在基于原型的方法中，使得不是每个新的产品系列都需要一个新的具体工厂类。

* 定义可扩展的工厂
  
  AbstractFactory通常为每一种它可以生产的产品定义一个操作。产品的种类被编码在操作型构中。增加一种新的产品要求改变AbstractFactory的接口以及所有与它相关的类。一个更灵活但不太安全的设计是给创建对象的操作增加一个参数。该参数指定了将被创建的对象的种类。它可以是一个类标识符、一个整数、一个字符串，或其他任何可以标识这种产品的东西。实际上使用这种方法，AbstractFactory只需要一个“Make”操作和一个指示要创建对象的种类的参数。这是前面已经讨论过的基于原型的和基于类的抽象工厂的技术。

  该方法即使不需要类型强制转换，但仍有一个本质的问题：所有的产品将返回类型所给定的*相同*的抽象接口返回给客户。客户将不能区分或对一个产品的类型进行安全的假定。如果一个客户需要进行与特定子类相关的操作，而这些操作却不能通过抽象接口得到。虽然客户可以实施一个向下类型转换（downcast），但这并不总是可行或安全的，因为向下类型转换可能会失败。这是一个典型的高度灵活和可扩展接口的权衡折中。

#### 10. 代码示例
我们将使用Abstract Factory模式创建我们在开篇中所讨论的迷宫。

类MazeFactory可以创建迷宫的组件。它建造房间、墙壁和房间之间的门。它可以用于一个从文件中读取迷宫说明图并建造相应的迷宫程序。或者它可以被用于一个随机建造迷宫的程序。建造迷宫的程序将MazeFactory作为一个参数，这样程序员就能知道要创建的房间、墙壁和门等类。
{% highlight java %}
public class MazeFactory {
    public MazeFactory(){

    };

    public Maze makeMaze(){
        return  new Maze();
    }

    public Room makeRoom(int roomNo){
        return new Room(roomNo);
    }

    public Wall makeWall(){
        return new Wall();
    }

    public Door makeDoor(Room r1, Room r2){
        return new Door(r1, r2);
    }
}
{% endhighlight %}
回想一下建立一个有两个房间和它们之间的门组成的小迷宫的成员函数CreateMaze。CreateMaze对类名进行硬编码，这是的很难用不同的组件创建迷宫。这里是一个以MazeFactory为参数的新版本的CreateMaze，它修改了这个缺点：
{% highlight java %}
public class MazeGame {
    public Maze createMaze(MazeFactory factory) {
        Maze aMaze = factory.makeMaze();
        Room r1 = factory.makeRoom(1);
        Room r2 = factory.makeRoom(2);
        Door aDoor = factory.makeDoor(r1, r2);

        aMaze.addRoom(r1);
        aMaze.addRoom(r2);

        r1.setSides(Direction.North, factory.makeWall());
        r1.setSides(Direction.East, aDoor);
        r1.setSides(Direction.North, factory.makeWall());
        r1.setSides(Direction.West, factory.makeWall());

        r2.setSides(Direction.North, factory.makeWall());
        r2.setSides(Direction.East, factory.makeWall());
        r2.setSides(Direction.North, factory.makeWall());
        r2.setSides(Direction.West, aDoor);

        return aMaze;
    }
}
{% endhighlight %}
我们创建MazeFactory的子类EnchantedMazeFactory，这是一个创建施了魔法的迷宫的工厂。EnchantedMazeFactory将重新定义不同的成员函数并返回Room，Wall等不同的子类。
{% highlight java %}
public class EnchantedMazeFactory extends MazeFactory {
    public EnchantedMazeFactory(){

    }

    public Room makeRoom(int roomNo){
        return new EnchantedRoom(roomNo, castSpell());
    }

    private Spell castSpell() {
        Spell spell = new Spell();
        return spell;
    }

    public Door makeDoor(Room r1, Room r2){
        return new DoorNeedingSpell(r1, r2);
    }
}
{% endhighlight %}
现在假设我们想生成一个迷宫游戏，在这个游戏里，每个房间中可以有一个炸弹。如果这个炸弹爆炸，它将（至少）毁坏墙壁。我们可以生成一个Room的子类以明了是否有一个炸弹在房间中已经该炸弹是否已经爆炸了。我们也将需要一个Wall的子类以明了对墙壁的损害。我们将称这些类为RoomWithABomb和BombedWall。我们将定义的最后一个类是BombedMazeFactory，它是MazeFactory的子类，保证了墙壁是BombedWall类的而房间是RoomWithABomb的。BombedMazeFactory仅需重定义两个函数：
{% highlight java %}
public class BombedMazeFactory extends MazeFactory {
    public BombedMazeFactory(){

    }

    public Room makeRoom(int roomNo){
        return new RoomWithABomb(roomNo);
    }

    public Wall makeWall(){
        return new BombedWall();
    }
}
{% endhighlight %}
CreateMaze可以接受不同的Factory来建造相应的迷宫。

注意MazeFactory仅是工厂方法的一个集合。这是最通常的实现Abstract Factory模式的方式。同事注意MazeFactory不是一个抽象类；因为它既作为AbstractFactory也作为ConcreteFactory。这是Abstract Factory模式的简单应用的另一个通常的实现。因为MazeFactory是一个完全由工厂方法组成的具体类，通过生成一个子类并重定义需要改变的操作，它很容易生成一个新的MazeFactory。

CreateMaze使用房间的SetSide操作以指定它们的各面。如果它用一个BombedMazeFactory创建房间，那么该迷宫将由有BombedWall面的RoomWithABomb对象组成。如果RoomWithABomb必须访问一个BombedWall的与特定子类相关的成员，那么它将不得不对它的墙壁引用进行从Wall到BombedWall的转换。只要该参数确实是一个BombedWall，这个向下转换就是安全的，而如果墙壁仅由一个BombedMazeFactory创建就可以保证这一点。

#### 11. 已知应用

#### 12. 相关模式
AbstractFactory类通常用工厂方法（Factory Method）实现，但它们也可以用Prototype实现。一个具体的工厂通常是一个单件（Singleton)。































