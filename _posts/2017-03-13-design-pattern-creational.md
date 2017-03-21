---
layout: post
title: "设计模式 - 创建型模式 - 开篇"
description: "抽象了实例化的过程"
tags: [design,pattern]
---

### 设计模式分类

设计模式在粒度和抽象层次上各不相同。我们可以根据两条准则对模式进行分类。第一是**目的**准则，即模式是用来完成什么工作的。模式依据其目的可以分为**创建型（Creational）、结构型（Structural)、行为型（Bahavioral)**三种。创建型模式与对象的创建有关；结构型模式处理类或者对象的组合；行为型模式对类或对象怎样交互和怎样分配职责进行描述。第二是**范围**准则，指定模式主要是用于类还是用于对象。类模式处理类和子类之间的关系，这些关系通过继承建立，是静态的，在编译时刻便确定下来了。对象模式处理对象间的关系，这些关系在运行时刻是可以变化的，更具动态性。从某种意义上来说，几乎所有模式都使用继承机制，所以“类模式”只指那些集中于处理类间关系的模式，而大部分模式都属于对象模式的范畴。

创建型类模式将对象的部分创建工作延迟到子类，而创建型对象模式则将它延迟到另一个对象中。结构型类模式使用继承机制来组合类，而结构型对象模式则描述了对象的组装方式。行为型类模式使用继承描述算法和控制流，而行为型对象模式则描述一组对象怎样协作完成单个对象所无法完成的任务。

### 创建型模式
创建型模式抽象了实例化过程。它们帮助一个系统独立于如何创建、组合和表示它的那些对象。一个类创建型模式使用继承改变被实例化的类，而一个对象创建型模式将实例化委托给另一个对象。

随着系统演化得越来越依赖于对象复合而不是类继承，创建型模式变得更为重要。当这种情况发生时，重心从一组固定行为的硬编码（hard-coding)转移为定义一个较小的行为集，这些行为可以被组合成任意数目的更复杂的行为。这样创建有特定行为 的对象要求的不仅仅是实例化一个类。

在这些模式中有两个不断出现的主旋律。第一，它们都将关于该系统使用哪些具体的类的信息封装起来。第二，它们隐藏了这些类的实例是如何被创建和放在一起的。整个系统关于这些对象所知道的是由抽象类所定义的接口。因此，创建型模式在*什么*被创建，*谁*创建它，它是*怎样*被创建的，以及*何时*创建这些方面给予你很大的灵活性。它们允许你用结构和功能差别很大的“产品”对象配置一个系统。配置可以是静态的（即在编译时指定），也可以是动态的（在运行时）。

有时创建型模式是相互竞争的。例如，在有些情况下Prototype或Abstract Factory用起来都很好。而在另外一些情况下它们是互补的：Builder可以使用其他模式去实现某个构建的创建。Prototype可以再它的实现中使用Singleton.

因为创建型模式紧密相关，我们将所有5个模式一起研究以突出它们的相似点和相异点。我们也将举一个通用的例子--为一个电脑游戏创建一个迷宫--来说明它们的实现。这个迷宫和游戏将随着各种模式不同而略有区别。有时这个游戏将仅仅是找到一个迷宫的出口；在这种情况下，游戏者可能仅能见到该迷宫的局部。有时迷宫包括一些要解决的问题和要战胜的危险，并且这些游戏可能会提供已经被探索过的那部分迷宫地图。

我们将忽略许多迷宫中的细节以及一个迷宫游戏中有一个还是多个游戏者。我们仅仅关注迷宫是怎样被创建的。我们将一个迷宫定义为一系列房间，一个房间知道它的邻居；可能的邻居要么是另一个房间，要么是一堵墙，或者是到另一个房间的一扇门。

类Room、Door和Wall定义了我们所有的例子中使用到的构件。我们仅定义这些类中对创建一个迷宫起重要作用的一些部分。下图表示这些类之间的关系：
{% include image.html path="documentation/design-pattern/Demo-Creational.png" path-detail="documentation/design-pattern/Demo-Creational.png" alt="Demo-Creational" %}

每个房间有四面，我们使用枚举类型Direction来指定房间的东南西北.
{% highlight java %}
public enum Direction {
    North, South, East, West
}
{% endhighlight %}
类MapSite是所有迷宫组件的公共抽象类。为了简化例子，MapSite仅定义了一个操作Enter，它的含义决定于你在进入什么。如果你进入一个房间，那么你的位置会发生改定。如果你试图进入一扇门，那么这两件事情中就有一件会发生：如果门是开着的，你进入另一个房间。如果门是关着的，那么你就会碰壁。
{% highlight java %}
public abstract class MapSite {
    public abstract void enter();
}
{% endhighlight %}

Enter为更加复杂的游戏操作提供了一个简单基础。例如，如果你在一个房间中说“向东走”，游戏只能确定直接在东边的是哪一个MapSite并对它调用Enter。特定子类的Enter操作将计算出你的位置是发生改变，还是你会碰壁。在一个真正的游戏中，Enter可以将移动的游戏者对象作为一个参数。

Room是MapSite的一个具体的子类，而MapSite定义了迷宫中构件之间的主要关系。Room有指向其他MapSite对象的引用，并保存一个房间号，这个数字用来标识迷宫中的房间。
{% highlight java %}
public class Room extends MapSite {
    private int roomNo;
    private HashMap<Direction, MapSite> sides;

    Room(int roomNo){

    }

    public MapSite getSides(Direction direction) {

        return sides.get(direction);
    }

    public void setSides(Direction direction, MapSite side) {
       sides.put(direction, side);
    }

    @Override
    public void enter() {

    }
}
{% endhighlight %}
下面的类描述了一个房间的每一面所出现的墙壁或门。
{% highlight java %}
public class Wall extends MapSite {
    public Wall(){

    }

    @Override
    public void enter() {

    }
}

public class Door extends MapSite {
    public Door(Room room1, Room room2){

    }

    public Room otherSideFrom(Room otherSideFromRoom){
        return null;
    }

    @Override
    public void enter() {

    }

    private Room room1;
    private Room room2;
    private boolean isOpen;
}
{% endhighlight %}
我们不仅需要知道迷宫的各部分，还要定义一个用来表示房间集合的Maze类，用RoomNo操作和给定的房间号，Maze就可以找到一个特定的房间。
{% highlight java %}
public class Maze {
    public Maze(){

    }
    void addRoom(Room room){

    }
    
    Room roomNo(int roomNo){
        return null;
    }
    
//    private Maze maze;
}
{% endhighlight %}
定义的另一个类是MazeGame，由它来创建迷宫。一个简单直接的创建迷宫的方法是使用一系列操作将构建增加到迷宫中，然后连接它们。
{% highlight java %}
public class MazeGame {
    public Maze createMaze(){
        Maze aMaze = new Maze();

        Room r1 = new Room(1);
        Room r2 = new Room(2);
        Door theDoor = new Door(r1, r2);

        aMaze.addRoom(r1);
        aMaze.addRoom(r2);

        r1.setSides(Direction.North, new Wall());
        r1.setSides(Direction.East, theDoor);
        r1.setSides(Direction.North, new Wall());
        r1.setSides(Direction.West, new Wall());

        r2.setSides(Direction.North, new Wall());
        r2.setSides(Direction.East, new Wall());
        r2.setSides(Direction.North, new Wall());
        r2.setSides(Direction.West, theDoor);

        return aMaze;
    }
}
{% endhighlight %}
考虑到这个函数所做的仅是创建一个有两个房间的迷宫，它是相当复杂的。显然有办法使它变得更简单。例如：Room的构造器可以提前用墙壁来初始化房间的每一面。但这仅仅是将代码移到其他地方。这个成员成员函数真正的问题不在于它的大小而在于它*不灵活*。它对迷宫的布局进行硬编码。改变布局意味着改变这个成员函数，或是重定义它--这意味着重新实现整个过程--或是对它的部分进行改变--这容易产生错误并且不利于重用。

创建型模式显示如何使得这个设计更*灵活*，但未必会更小。特别是，它们将便于修改定义一个迷宫构件的类。假设你想在一个包含（所有的东西）施了魔法的迷宫的新游戏中重用一个已有的迷宫布局。施了魔法的迷宫游戏有新的构件，像DoorNeedingSpell，它是一扇仅随着一个咒语才能被锁上和打开的门；以及EnchantedRoom，一个可以有不寻常东西的房间，比如魔法钥匙或是咒语。你怎样才能较容易的改变CreateMaze以让它用这些新类型的对象创建迷宫呢？

这种情况下，改变的最大障碍是对被实例化的类进行硬编码。创建型模式提供了多种不同方法从实例化它们的代码中除去对这些具体类的显式引用：
* 如果CreateMaze调用虚函数而不是构造器来创建它需要的房间、墙壁和门，那么你可以创建一个MazeGame的子类并重新定义这些虚函数，从而改变被实例化的类。这一方法是Factory Method模式的一个例子。
* 如果传递一个对象给CreateMaze作参数来创建房间、墙壁和门，那么你可以传递不同的参数来改变房间、墙壁和门的类。这是Abstract Factory模式的一个例子。
* 如果传递一个对象给CreateMaze，这个对象可以在它所建造的迷宫中使用增加房间、墙壁和门的操作，来全面创建一个新的迷宫，那么你可以使用继承来改变迷宫的一些部分或该迷宫的建造的方式。这是Builder模式的一个例子。
* 如果CreateMaze由多种原型的房间、墙壁和门对象参数化，它拷贝并将这些对象增加到迷宫中，那么你可以用不同的对象替换这些原型对象以改变迷宫的构成。这是Prototype模式的一个例子。

剩下的创建型模式，Singleton，可以保证每个游戏中仅有一个迷宫而且所有的游戏对象都可以迅速访问它--不需要求助于全局变量或函数。Singleton也使得迷宫易于扩展或替换，且不需要变动已有的代码。