---
layout: post
title: "设计模式 - 创建型模式 - 工厂方法"
description: "定义一个用于创建对象的接口，让子类决定实例化哪一个类"
tags: [design,pattern]
---

### 对象创建型模式 -- 工厂方法（Factory Method）
{:.no_toc}

* 目录
{:toc}

#### 1. 意图
定义一个用于创建对象的接口，让子类决定实例化哪一个类。Factory Method使一个类的实例化延迟到子类。
#### 2. 别名
虚构造器（Virtual Constructor）
#### 3. 动机
框架使用抽象类定义和维护对象之间的关系。这些对象的创建通常也是由框架负责。

考虑这样一个应用框架，它可以向用户显示多个文档。这这个框架中，两个主要的抽象是类Application和Document。这两个类都是抽象的，客户必须通过它们的子类来做与具体应用相关的实现。例如：为了创建一个绘图应用，我们定义类DrawingApplication和DrawingDocument。Application类负责管理Document并根据需要创建它们--例如，当用户从菜单中选择Open或New的时候。

因为被实例化的特定Document子类是与特定应用相关的，所以Application类不可能预测到哪个Document子类将被实例化--Application类仅知道一个新的文档*何时*应被创建，而不知道*哪一种*Document将被创建。这就产生了一个尴尬的局面：框架必须实例化类，但是它只知道不能被实例化的抽象类。

Factory Method模式提供了一个解决方案。它封装了哪一个Document子类将被创建的信息并将这些信息从该框架中分离出来，如图示：
{% include image.html path="documentation/design-pattern/Factory-Method-Demo.png" path-detail="documentation/design-pattern/Factory-Method-Demo.png" alt="Factory-Method-Demo" %}
Application的子类重定义Application的抽象操作CreateDocument以返回适当的Document子类对象。一旦一个Application子类实例化以后，它就可以实例化与应用相关的文档，而无需知道这些文档的类。我们称CreateDocument是一个工厂方法（factory method），因为它负责“生产”一个对象。
#### 4. 适用性
在下列情况下可以使用Factory Method模式：
* 当一个类不知道它所必须创建的对象的类的时候。
* 当一个类希望由它的子类来指定它所创建的对象的时候。
* 当类将创建对象的职责委托给多个帮助子类中的某一，并且你希望将哪一个帮助子类是代理者这一信息局部化的时候。

#### 5. 结构
{% include image.html path="documentation/design-pattern/Factory-Method-Pattern-Class-Diagram.png" path-detail="documentation/design-pattern/Factory-Method-Pattern-Class-Diagram.png" alt="Factory-Method-Pattern-Class-Diagram
" %}

#### 6. 参与者
* Product（Document）
  - 定义工厂方法所创建的对象的接口
* ConcreteProduct（MyDocument）
  - 实现Product接口
* Creator（Application）
  - 声明工厂方法，该方法返回一个Product类型的对象。Creator也可以定义一个工厂方法的缺省实现，它返回一个缺省的ConcreteProduct对象。
  - 可以调用工厂方法以创建一个Product对象。
* ConcreteCreator（MyApplication）
  - 重定义工厂方法以返回一个ConcreteProduct实例
  
#### 7. 协作
Creator依赖于它的子类来定义工厂方法，所以它返回一个适当的ConcreteProduct实例。

#### 8. 效果
工厂方法不再将与特定应用有关的类绑定到你的代码中。代码仅处理Product接口；因此它可以与用户定义的任何ConcreteProduct类一起使用。

工厂方法的一个潜在缺点在于客户可能仅仅为了创建一个特定的ConcreteProduct对象，但是就不得不创建Creator子类。当Creator子类不必须时，客户现在必然要处理类演化得其他方面；但是当客户无论如何必须创建Creator的子类时，创建子类也是可行的。

下面是Factory Method模式的另外两种效果：
* 为子类提供挂钩（hook）

  用工厂方法在一个类的内部创建对象通常比直接创建的对象更灵活。Factory Method给子类一个挂钩以提供对象的扩展版本。
  在Document的例子中，Document类可以定义一个称为CreateFileDialog的工厂方法，该方法为打开一个已有的文档创建默认的文件对话框对象。Document子类可以重定义这个工厂方法以定义一个与特定应用相关的文件对话框。在这种情况下，工厂方法就不再抽象了而是提供了一个合理的缺省实现。

* 连接平行的类层次

  迄今为止，在我们所考虑的例子中，工厂方法并不仅仅被Creator调用，客户可以找到一些有用的工厂方法，尤其在平行类层次的情况下。
  当一个类将它的一些职责委托给一个独立的类的时候，就产生了类平行层次。考虑可以被交互操纵的图形；也就是说，它们可以用鼠标进行伸展、移动，或者旋转。实现这样一些交互并不总是那么容易，它通常需要存储和更新在给定时刻记录操纵状态的信息。这个状态仅仅在操纵时需要。因此它不需要被保存在图形对象中。此外，当用户操纵图形时，不同的图形有不同的行为。例如，将直线图形拉长可能会产生一个端点被移动的效果，而伸展正文图形则可能会改变行距。

  有了这些限制，最好使用给一个独立的Manipulator对象实现交互并保存所需要的任何与特定操纵相关的状态。不同的图形将使用不同的Manipulator子类来处理 特定的交互。得到的Manipulator类层次与Figure类层次是平行（至少部分平行），如下图所示：
  {% include image.html path="documentation/design-pattern/Demo-Parallel-Class-Hierarchy.png" path-detail="documentation/design-pattern/Demo-Parallel-Class-Hierarchy.png" alt="Demo-Parallel-Class-Hierarchy" %}
  Figure类提供了一个CreateManipulator工厂方法，它使得客户可以创建一个与Figure相对应的Manipulator。Figure子类重定义该方法以返回一个合适的Manipulator子类实例。做为一种选择，Figure类可以实现CreateManipulator以返回一个默认的Manipulator实例，而Figure子类可以只是继承这个缺省实现。这样的Figure类不要相应的Manipulator子类--因此该层次只是部分平行。

  `注意`工厂方法是怎样定义两个类层次之间的连接的。它将哪些类应一同工作的信息局部化了。

#### 9. 实现
当应用Factory Method模式时要考虑下面一些问题：
* 主要有两种不同的情况

  Factory Method模式主要有两种不同的情况：1）Creator类是一个抽象类并且不提供它所声明的工厂方法的实现。2）Creator是一个具体的类而且为工厂方法提供一个缺省的实现。也有可能有一个定义了缺省实现的抽象类，但这不太常见。

  第一种情况需要子类来定义实现，因为没有合理的缺省实现。它避免了不得不实例化不可以预见类的问题。第二种情况中，具体的Creator主要因为灵活性才使用工厂方法。它所遵循的准则是，“用一个独立的操作创建对象，这样子类才能重定义它们的创建方式。”这条准则保证了子类的设计者能够在必要的时候改变父类所实例化的对象的类。

* 参数化工厂方法
 
  该模式的另一种情况使得工厂方法可以创建多种产品。工厂方法采用一个标识要被创建的对象种类参数。工厂方法创建的所有对象将共享Product接口。在Document的例子中个，Application可能支持不同种类的Document。你给CreateDocument传递一个外部参数来指定将要创建的文档的种类。

* 特定语言的变化问题
  
  不同的语言有助于产生一些有趣的变化和警告（caveat）。

* 使用模板以避免创建子类
  正如我们已经提及的，工厂方法另一个潜在的问题是它们可能仅为了创建适当的Product对象而迫使你创建一个Creator子类。在C++中另一个解决办法是提供Creator的一个模板子类，它使用Product类作为模板参数。

* 命名约定

  使用命名约定是一个好习惯，它可以清楚地说明你正在使用工厂方法。例如，Macintosh的应用框架MacAPP总是声明那些定义为工厂方法的抽象操作为Class*DoMakeClass(), 此处Classshi Product类。

#### 10. 代码示例
函数CreateMaze建造并返回一个迷宫。这个函数存在的一个问题是它对迷宫、房间、门和墙壁的类进行了硬编码。我们将引入工厂方法以使子类可以徐泽这些构建。首先我们将在MazeGame中定义工厂方法以创建迷宫、房间、和门对象:
{% highlight java %}
public class MazeGame {
    public Maze createMaze(){
        //...
    }

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
每一个工厂方法返回一个给定类型的迷宫构件。MazeGame提供一些缺省的实现，它们返回最简单的迷宫、房间、墙壁和门。
{% highlight java %}
public class MazeGame {
    public Maze createMaze(){
        Maze aMaze = new Maze();

        Room r1 = makeRoom(1);
        Room r2 = makeRoom(2);
        Door theDoor = makeDoor(r1, r2);

        aMaze.addRoom(r1);
        aMaze.addRoom(r2);

        r1.setSides(Direction.North, makeWall());
        r1.setSides(Direction.East, theDoor);
        r1.setSides(Direction.North, makeWall());
        r1.setSides(Direction.West, makeWall());

        r2.setSides(Direction.North, makeWall());
        r2.setSides(Direction.East, makeWall());
        r2.setSides(Direction.North, makeWall());
        r2.setSides(Direction.West, theDoor);

        return aMaze;
    }
    //...
}
{% endhighlight %}
不同的游戏可以创建MazeGame的子类以特别指明一些迷宫的部件。MazeGame子类可以重定义一些或所有的工厂方法以指定产品中的变化。例如，一个BombedMazeGame可以重新定义产品Room和Wall以返回爆炸后的变体。
{% highlight java %}
public class BombedMazeGame extends MazeGame {
    public BombedMazeGame() {

    }

    @Override
    public Wall makeWall(){
        return new BombedWall();
    }

    @Override
    public Room makeRoom(int roomNO){
        return new RoomWithABomb(roomNO);
    }
}
{% endhighlight %}

#### 11. 已知应用
工厂方法主要用于工具包和框架中。

#### 12. 相关模式
Abstract Factory经常用工厂方法来实现。我们将在Abstract Factory模式的动机中说明。

工厂方法通常在Template Methods中被调用。

Prototype不需要创建Creator的子类。但是，它们通常要求一个针对Product类的Initialize操作。Creator使用Initialize来初始化对象。而Factory Method不需要这样的操作。