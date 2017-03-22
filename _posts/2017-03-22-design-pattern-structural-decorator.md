---
layout: post
title: "设计模式 - 结构型模式 - 装饰"
description: "动态地给一个对象添加一些额外的职责"
tags: [design,pattern]
---

### 对象结构型模式 - 装饰（DECORATOR）

#### 1. 意图
动态地给一个对象添加一些额外的职责。就增加功能来说，Decorator模式比生产子类更为灵活。

#### 2. 别名
包装器Wrapper

#### 3. 动机
有时我们希望给某个对象而不死整个类添加一些功能。例如，一个图形用户界面工具箱允许你对任意一个用户界面组件添加一些特性，例如边框，或是一些行为，例如窗口滚动。

使用继承机制是添加功能的一种有效途径，从其他类继承过来的边框特性可以被多个子类的实例所使用。但这个方法不够灵活，因为边框的选择是静态的，用户不能控制对组件加边框的方式和时机。

一种较为灵活的方式是将组件嵌入另一个对象中，由这个对象添加边框。我们称这个嵌入的对象为**装饰**。这个装饰与所有的装饰的组件接口一致，因此它对使用该组件的客户透明。它将客户请求转发给该组件，并且可能在转发前后执行一些额外的动作（例如画一个边框）。透明性使得你可以递归的嵌套多个装饰，从而可以添加任意多的功能。

例如，假定有一个对象TextView，它可以再窗口中显示正文。缺省的TextView没有滚动条，因为我们可能有时并不需要滚动条。当需要滚动条时，我们可以用ScrollDecorator添加滚动条。如果我们还想再TextView周围添加一个粗黑边框，可以使用BorderDecorator添加。因此只要简单地将这些装饰和TextView进行组合，就可以达到预期的效果。 
{% include image.html path="documentation/design-pattern/Decorator-Composite-Demo.png" path-detail="documentation/design-pattern/Decorator-Composite-Demo.png" alt="Decorator-Composite-Demo" %}

ScrollDecorator和BorderDecorator类是Decorator类的子类。Decorator类是一个可视组件的抽象类，用于装饰其他可视组件，如下图所示。
{% include image.html path="documentation/design-pattern/Decorator-Demo.png" path-detail="documentation/design-pattern/Decorator-Demo.png" alt="Decorator-Demo" %}
VisualComponent是一个描述可视对象的抽象类，它定义了绘制和事件处理的接口。`注意`Decorator类怎样将绘制请求简单地发送给它的组件，以及Decorator的子类如何扩展这个操作。

Decorator的子类为特定功能可以自由地添加一些操作。例如，如果其他对象知道界面中恰好有一个ScrollDecorator对象，这些对象就可以用ScrollDecorator兑现搞得ScrollTo操作滚动这个界面。这个模式中有一点很重要，它使得在VisualComponent可以出现的任何地方都可以有装饰。因此，客户通常不会感觉到装饰过的组件与为装饰组件之间的差异，也不会与装饰产生任何依赖关系。

#### 4. 适用性
* 在不影响其他对象的情况下，以动态、透明的方式给对个对象添加职责。
* 处理那些可以撤销的职责。
* 当不能采用生成子类的方法进行扩充时。一种情况是，可能有大量独立的扩展，为支持每一种组合将产生大量的子类，使得子类数目呈爆炸式增长。另一种情况可能是因为类定义被隐藏，或类定义不能用于生成子类。

#### 5. 结构
{% include image.html path="documentation/design-pattern/Decorator-Pattern-Structure.png" path-detail="documentation/design-pattern/Decorator-Pattern-Structure.png" alt="Decorator-Pattern-Structure" %}

#### 6. 参与者
* Component（VisualComponent）  定义一个对象接口，可以给这些对象动态地添加职责。
* ConcreteComponent（TextView）  定义一个对象，可以给这个对象添加一些职责。
* Decorator  维持一个指向Component对象的指针，并定义一个与Component接口一致的接口。
* ConcreteDecorator（BorderDecorator，ScrollDecorator）  向组件添加职责。

#### 7. 协作
Decorator将请求转发给它的Component对象，并有可能在转发请求前后执行一些附加的动作。

#### 8. 效果
Decorator模式至少有两个主要的`优点`与`缺点`：
* 比静态继承更灵活  

  与对象的静态继承（多重继承）相比，Decorator模式提供了更加灵活的向对象添加职责的方式。可以用添加和分离的方法，用装饰在运行时候增加和删除职责。相比之下，继承机制要求为每个添加的职责创建一个新的子类（例如，BorderScrollableTextView，BorderedTextView)。这会产生许多新的类，并且会增加系统的复杂度。此外，为一个特定的Component类提供多个不同的Decorator类，这就使得你可以对一些职责进行混合和匹配。使用Decorator模式可以很容易地重复添加一个特性，例如在TextView上添加双边框时，仅需将添加两个BorderDecorator即可。而两次继承Border类则极容易出错。

* 避免在层次结构高层的类有太多的特性  

  Decorator模式提供了一种“即用即付”的方法来添加职责。它并不试图在一个复杂的可定制的类中支持所有可预见的特征，相反，你可以定义一个简单的类，并且用Decorator类给它逐渐地添加功能。可以从简单的部件组合出复杂的功能。这样，应用程序不必为不需要的特征付出代价。同时也更易于不依赖于Decorator所扩展（甚至是不可预知的扩展）的类而独立地定义新类型的Decorator。扩展一个复杂类的时候，很可能会暴露与添加的职责无关的细节。

* Decorator与它的Component不一样

  Decorator是一个透明的包装。如果我们从对象标识的观点出发，一个被装饰了的组件与这个组件是有差别的，因此，使用装饰时不应该依赖对象标识。

* 有许多小对象  

  采用Decorator模式进行系统设计往往会产生许多看上去类似的小对象，这些对象仅仅在他们相互连接的方式上有所不同，而不是它们的类或是它们的属性值有所不同。尽管对于那些了解这些系统的人来说，很容易对它们进行定制，但是很难学习这些系统，排错也很困难。

#### 9. 实现
使用Decorator模式时应注意一下几点：
* `接口的一致性`  装饰对象的接口必须与它所装饰的Component的接口是一致的，因此，所有的ConcreteDecorator类必须有一个公共的父类（至少在C++中如此）。

* `省略抽象的Decorator类`  当你仅需要添加一个职责时，没有必要定义抽象Decorator类。你常常需要处理现存的类层次而不是设计一个新系统，这时你可以把Decorator向Component转发请求的责任合并到ConcreteDecorator中。

* `保持Component类的简单性`  为了保证接口的一致性，组件和装饰必须有一个公共的Component父类。因此保持这个类的简单性是很重要的；即，它应集中于定义接口而不是存储数据。对数据表示的定义应延迟到子类中，否则Component类会变得过于复杂和庞大，因而难以大量使用。赋予Component太多的功能也使得，具体的子类有一些它们并不需要的功能的可能性大大增加。

* `改变对象外壳与改变对象内核` 我们可以将Decorator看作一个对象的外壳，它可以改变这个对象的行为。另外一种方法是改变对象的内核。例如，Strategy模式就是一个用于改变内核的很好的模式。当Component类原本就很庞大时，使用Decorator模式代价太高，Strategy模式相对更好一些。在Strategy模式中，组件将它的一些行为转发个一个独立的策略对象，我们可以替换strategy对象，从而改变或扩充组件的功能。

#### 10. 代码示例
我们假定已经存在一个Component类VisualComponent。
{% highlight java %}
public abstract class VisualComponent {
    public VisualComponent(){

    };

    abstract void draw();
    abstract void resize();
}
{% endhighlight %}

定义一个VisualComponent的一个子类Decorator，我们将生成Decorator的子类以获取不同的装饰。Decorator装饰由_component实例变量引用的VisualComponent，这个实例变量在构造器中被初始化。对于VisualComponent接口中定义的每一个操作，Decorator类都定义了一个缺省的实现，这一实现将请求转发给_component。
{% highlight java %}
public abstract class Decorator extends VisualComponent {
    private VisualComponent _component;

    public Decorator(VisualComponent component){
        this._component = component;
    }

    void draw(){
        _component.draw();
    }

    void resize(){
        _component.resize();
    }
}
{% endhighlight %}

Decorator的子类定义了特殊的装饰功能，例如，BorderDecorator类为它所包含的组件添加了一个边框。BorderDecorator是Decorator的子类，它重定义Draw操作用于绘制边框。同时BorderDecorator还定义了一个私有的辅助操作DrawBorder，由它绘制边框。这些子类继承了Decorator类所有其他的操作。
{% highlight java %}
public class BorderDecorator extends Decorator {
    private int _width;

    public BorderDecorator(VisualComponent component, int borderWidth){
        super(component);
        this._width = borderWidth;

    }

    void draw(){
        super.draw();
        drawBorder(_width);
    }

    private void drawBorder(int width){
        //do something
    }
}
{% endhighlight %}

类似的可以实现DropShadowDecorator，它们给可视组件添加滚动和阴影功能。现在我们组合这些类的实例以提供不同的装饰效果，一下代码展示了如何使用Decorator创建一个具有边界的可滚动TextView。首先将一个可视组件放入窗口对象中，假设Window类已经提供了setContents操作
{% highlight java %}
Window window = new Window();
TextView textView = new TextView();

window.setContents(textView); 

//如果要添加一个有边界和可以滚动的TextView
window.setContents(new BorderDecorator(
    new ScrollDecorator(textView), 1)
    );
{% endhighlight %}
由于Window通过VisualComponent接口访问它的内容，因此它并不知道存在该装饰。如果你需要直接与正文视图交互，例如，你想调用一些操作，而这些操作不是VisualComponent接口的一部分，此时你可以跟踪正文视图。依赖于组件标识的客户也应该直接应用它。

#### 11. 已知应用
许多面向对象的用户界面工具箱使用装饰为你窗口组件添加图形装饰。但是Decorator模式不仅仅局限于图形用户界面，下面的例子说明了这一点。

Streams是大多数I/O设备的基础抽象结构，它提供了将对象转换成字节或字符流的操作接口，使我们可以将一个对象转变成一个文件或内存中的字符串，可以再以后恢复使用。一个简单直接的方法是定义一个抽象的Stream类，它有两个子类MemoryStream与FileStream。但是假定我们还希望能够做下面一些事情：
* 用不同的压缩算法对数据流进行压缩。
* 将数据简化为7位ASCII码字符，这样它就可以在ASCII信道上传输。

下图的给出了一个解决问题的方法。
{% include image.html path="documentation/design-pattern/Decorator-Application-Stream.png" path-detail="documentation/design-pattern/Decorator-Application-Stream.png" alt="Decorator-Application-Stream" %}
{% highlight java %}
Stream aStream = new CompressingStream(
    new ASCIIStream(
        new FileStream("aFileName"))
    );

aStream.putInt(12);
aStream.putString("aString");
{% endhighlight %}

#### 12. 相关模式
Adapter模式：Decorator模式不同于Adapter模式，因为装饰仅改变对象的职责而不改变它的接口；而适配器将给对象一个全新的接口。

Composite模式：可以将装饰视为一个退化的、仅有一个组件的组合。然而，装饰仅给对象添加一些额外的职责 -- 它的目的不在于对象聚集。

Strategy模式：用一个装饰你可以改变对象的外表；而Strategy模式使你可以改变对象的内核。这是改变对象的两种途径。