---
layout: post
title: "单元测试代码指导规范"
description: "如何写出优美而实效的单元测试代码"
tags: [design, code]
---
大家在写代码的时候，很多人很多时候都一直在提及关于测试，关于单元测试之于产品质量的重要性，我一直也想找个机会给自己做个关于Unit Testing方面的总结，在网上看到了一篇自己觉得质量很高，也很符合自己期望的文章，翻译在这里供有志趣的同路人们一起把玩。

### 什么是单元测试（Unit Testing）？
> 单元测试又称为模块测试, 是针对程序模块（软件设计的最小单位）来进行正确性检验的测试工作。程
> 序单元是应用的最小可测试部件。在过程化编程中，一个单元就是单个程序、函数、过程等；对于面向
> 对象编程，最小单元就是方法，包括基类（超类）、抽象类、或者派生类（子类）中的方法。

### 单元测试代码指导规范

#### 1. 保持单元测试代码短小而快速
理想情况下，在每次代码迁入之前都要执行整个测试集。保持测试代码能够快速跑完以减少代码开发的周转期

#### 2. 单元测试代码应该全自动化并且没有相互影响
测试集一般在一个定期执行的系统里面，必须是全自动化的，并且是有用的。如果执行的结构需要人工审查，那个这些测试不应该出现在单元测试里面。

#### 3. 单元测试必须很简单就能运行
配置开发环境使得单一的测试和测试集能通通过一个单一的命令或者按一个按键就能跑起来。

#### 4. 测试结果能够衡量
将覆盖率分析应用到单元测试上，这样可以看到测试执行的精确结果并且可以调查哪部分的代码有没有执行到。

#### 5. 立即修复失败的测试
每个开发人员都要保证新的测试能够，并且所有的已有的测试在你的代码迁入后能跑成功。

#### 6. 保持测试是在单元级别
单元测试是用来测试类。每个普通的类都应该有一个测试类，这个类的行为应该被独立的测试。避免通过单元测试的框架去测试整个工作流程的尝试，这样的测试是很慢并且难以维护。工作流程的测试应该放在它们应该出现的地方，但是绝不是单元测试，它们必须独立的设置与执行。

#### 7. 从简单开始
一个简单的测试毫无疑问的好于完全没有测试。一个简单的测试类可以建立起目标类的测试架构，它能去验证构建环境，单元测试环境，执行环境以及覆盖率分析工具的表现和正确性。它能够证实目标类是整个装载的一部分并且能够访问。Hello，World！的单元测试可以是这样的：
{% highlight html %}
     void testDefaultConstruction()
     {
       Foo foo = new Foo();
       assertNotNull(foo);
     }
{% endhighlight %}

#### 8. 保持单元测试代码的独立性
为了保证测试的简化性和维护起来简单，测试代码绝不应该依赖其他测试代码或者是测试代码执行的顺序。

#### 9. 保证测试代码接近于被测试的类
如果被测试的类是*Foo*,那么测试类应该叫*FooTest*（不是*TestFoo*），与*Foo*的包结构也保持一致。保持测试代码类在一个分离的目录结构中使它们访问与维护起来更难。确保构建环境是可配置的，这样测试类就不会进行产品代码库或者在产品代码中执行。

#### 10. 恰当的命名测试
保证每一个测试方法测试类的一个区域的功能，并且给测试方法相应的命名。典型的命名惯例是测试【什么】，如：*testSaveAs()*, *testAddListener()*, *testDeleteProperty()*等等。

#### 11. 测试开发的应用程序接口
单元测试可以通过它们的公共的API定义成测试类。一个测试工具使得类的私有内容的测试变成可能，但是这个应该避免当它把测试变得更加冗余和维护起来更加困难时。如有有一些私有内容看上去需要明确的测试，可以考虑把它们重构成公共方法放到公用的类里面去。但是做这个的目的是提升设计的通用性，而不是为了辅助测试。

#### 12. 黑盒思考
把自己当做第三方类的消费者，测试这个类是否满足它的需求，始终找出问题。

#### 13. 白盒思考
毕竟，测试程序员同样写被测试的类，额外的努力应该放到去测试最复杂的逻辑。

#### 14. 琐碎的cases同样需要测试
有时推荐所有的重要的cases都需要测试，而一些琐碎的方法像什么简单的*setters*和*getters*可以忽略。然后，这里有若干个原因说明为什么琐碎的cases也需要测试：首先，琐碎很难去界定，不同的东西对不同的人是不同的。从黑盒的角度看，没办法知道代码的哪个部分是琐碎的。同样，琐碎的代码中也包含错误，经常由*copy-paste*所导致。
{% highlight markdown %}
     private double weight_;
     private double x_, y_;
     public void setWeight(int weight)
     {
       weight = weight_;  // error
     }
     public double getX()
     {
       return x_;
     }
    public double getY()
     {
       return x_;  // `error`
     }
{% endhighlight %}
因此建议所有的代码都要测试，毕竟，琐碎的代码很好测试。

#### 15. 首先专注于执行的覆盖率
将执行覆盖率与实际的测试覆盖率区分开。一个测试的最基本的目标应该保证好的执行覆盖率。这将保证代码在一些输入参数下确实是被执行了。如果做个了这个，那么测试覆盖率就会提升。注意，实际的测试覆盖率很难去衡量（并且总是接近0%）。考虑下面的公共方法：
{% highlight markdown %}
     void setLength(double length);
{% endhighlight %}
调用*setLength(1.0)*你可以获得100%的执行覆盖率。为了获得100%的实际测试覆盖率，这个方法必须在输入每一个可能的*double*值时被调用，并且这些输入后的每一次调用结果都要验证通过。这绝对不可能办到！

#### 16. 覆盖边缘cases
确保参数的边缘cases被覆盖到。对于数字，要测试负数，0，正数，最小值，最大值，NaN，无穷，等等。对于字符串，要测试空字符串，单个字符字符串，non-ACSII字符串，multi-MB字符串，等等。对于集合，测试空集合，一个，第一个，最后一个，等等。对于日期，测试1月1日，2月29日，12月31日，等等。被测试的类需要建议每个具体case的边缘cases。这样是为了确保尽可能多的这些case能够被恰当的测试到，一般错误就在这是cases中。

#### 17. 提供一个随机值生成器
当边缘cases被覆盖到了，一个简单的进一步提升测试覆盖率的办法是生成随机值参数，这样测试每次执行的时候有不同的输入。为了达到这个目的，提供一个简单的工具类来这生成这些基本类型的随机值，像*double*，*integers*, *strings*,*date*等等。生成器能够生成整个域中的每一个类型。

如果测试跑起来很快的话，可以考虑把它们放在一个循环里面跑，这样可以跑尽可能多的输入组合。下面的例子验证了在小的和大的元组排列之间做两次转换，然后把值重新设回去后比较值是否相等。因为这个测试跑的很快，所有每次可以执行一百万个不同的值。
{% highlight markdown %}
    void testByteSwapper()
    {
      for (int i = 0; i < 1000000; i++) {
        double v0 = Random.getDouble();
        double v1 = ByteSwapper.swap(v0);
        double v2 = ByteSwapper.swap(v1);
        assertEquals(v0, v2);
      }
    }
{% endhighlight %}

#### 18. 每次测试一个功能
在测试模式下，有时会试着在每一个测试里面断言“所有的事情”。这是应该避免的，它会让测试变得难以维护。就只测试安装测试方法名所指示的功能。对一般的代码来说，目标是保持测试代码数量尽可能的少。

#### 19. 使用明确的断言
总是习惯用assertEquals(a, b)而不是ssertTrue(a == b)（反之亦然）作为前者能够提供更加有用的信息，在测试失败时更容易判断到底是哪里出错了。这个在上面类似的例子中，用随机值做参数时并且不知道这些参数是什么时候输入的时候显得尤为重要。

#### 20. 提供反面测试
反面测试故意错用代码来验证系统的健壮性和错误是否处理恰当。看下面的方法，如果输入负数作为参数来请求这个方法会抛出异常来：
{% highlight markdown %}
  void setLength(double length) throws IllegalArgumentException;
{% endhighlight %}
测试这个特殊case的正确行为可以这样做：
{% highlight markdown %}
  try {
    setLength(-1.0);
    fail();  // If we get here, something went wrong
  }
  catch (IllegalArgumentException exception) {
    // If we get here, all is fine
  }
{% endhighlight %}

#### 21. 在设计代码的时候就要考虑测试
单元测试的写与维护都是有代价的，最小化公共API和减少圈复杂度都是能降低代价、做出快速运行高覆盖率的测试代码的办法，同时也能让测试代码更易维护。
一些建议：
* 在构建是通过建立状态使得类成员不可改变。这个可以减少setter方法的需要。
* 限制过度继承的使用以及纯虚公共方法。
* 通过利用友元类或者包范围限制来减少公共API。
* 避免不必要的分支。
* 在分支里保持尽可能少的代码。
* 多使用exceptions和assertion来验证公共或私有API中的参数。
* 验证简便方法的使用。从黑盒的角度来看，每个方法必须做同等的测试。下面的一个小的实例：
{% highlight markdown %}
  public void scale(double x0,double y0,double scaleFactor)
  {
      // scaling logic
  }
  public void scale(double x0, double y0)
  {
        scale(x0, y0, 1.0);
  }
{% endhighlight %}
后面一个方法的测试很简单，但给客户端调用的留下了额外的负担。

#### 22. 不要连接提前定义的外面资源
单元测试不应该依赖于某些确定的环境上下文，它们可执行的条件也不应该被限制在这些确定的环境中。它们应该是在任何时候任何环境下都可以跑的。为了提供测试必须的资源，这些资源应该由测试自身来保证它们的可用性。比如，一个用来解析特定类型文件的类，不是从某些提前定义的位置取一个示例文件，而是把文件内容放到测试里面，在测试设置过程中，把它写到一个临时的文件里面，测试跑完了再把它删掉。

#### 23. 了解测试的花费
不写单元测试的是有代价的，但是写单元测试也是有代价的。两者之间需要一个平衡，就执行覆盖率来说，一般的业界标准是80%。
一般很难做到100%执行覆盖的地方是在与外部资源相关的错误与一次处理。要模拟在一次事务执行中间的一次database中断还是很可能的，相比较来说这个可能代价太高以至于用另一个办法-额外的代码审查来应对这个问题。

#### 24. 给测试排优先级
单元测试是一个典型的自下而上的过程，如果没有足够的资源来测试系统的各个方面，优先权应该给予更加底层的东西。

#### 25. 准备失败的测试代码
比如：
{% highlight markdown %}
  Handle handle = manager.getHandle();
  assertNotNull(handle);

  String handleName = handle.getName();
  assertEquals(handleName, "handle-01");
{% endhighlight %}
如果第一个assertion是false, 在随后的声明里面代码会中断，剩下的测试都不会执行。总是准备测试失败的代码，这样一个单一测试的失败不会带来整个测试集的失败。一般来说可以像下面这样重写：
{% highlight markdown %}
  Handle handle = manager.getHandle();
  assertNotNull(handle);
  if (handle == null) return;

  String handleName = handle.getName();
  assertEquals(handleName, "handle-01");
{% endhighlight %}

#### 26. 写可以重现bugs的测试
当发现了一个bug,写个测试来重现这个bug(如，一个失败的测试)，用这个测试来作为这个bug被成功解决的标准。

#### 27. 了解单元测试的局限性
单元测试不能用来证明代码的正确性！
一个失败的单元测试可以意味着代码中有错误，但是一个成功的测试什么也证明不了。

单元测试最为有用的应用是验证和底层需求的记录，以及回归测试中：确保代码在演进和重构的过程中保证实现的逻辑稳定。

因此单元测试不能取代恰当的预先设计和开发流程。单元测试应该被用做对已经建立起来的开发方法论的有价值的补充。

[`原文:`Unit Testing Guidelines](http://geosoft.no/development/unittesting.html)

References

[1] Unit test definition from Wikipedia:
http://en.wikipedia.org/wiki/Unit_testing

[2] A short description of white-box and black-box testing
http://www.faqs.org/faqs/software-eng/testing-faq/section-13.html

[3] Our favourite test framework for C++: CxxTest
http://cxxtest.sourceforge.net/

[4] Our favourite test framework for Java: TestNG
http://testng.org/

[5] Our favourite coverage analysis tool for C++: LCOV
http://ltp.sourceforge.net/coverage/lcov.php

[6] Our favourite coverage analysis tool for Java: Cobertura
http://cobertura.sourceforge.net/

[7] More arguments for not connecting to external resources:
http://www.artima.com/weblogs/viewpost.jsp?thread=126923

[8] A few unit testing recommendations from Apple:
http://developer.apple.com/documentation/DeveloperTools/Conceptual/UnitTesting/Articles/UTGuidelines.html

[9] Some JUnit best practices:
http://www.javaworld.com/javaworld/jw-12-2000/jw-1221-junit_p.html