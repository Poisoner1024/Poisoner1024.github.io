---
layout: post
title: "在代码审查中我们究竟想发现什么？- 性能篇"
description: "性能是压死骆驼的草堆"
tags: [web, develop]
---
这是关于`在代码审查中我们究竟想发现什么?`系列6篇文章中的第3篇。

在所有的架构/设计领域，非功能性的系统性能需求都要放在前期去考虑。无论你工作在低延时（low-latency）的要求响应时间在毫微秒级的交易系统，你创建了一个在线购物网站需要响应用户请求；还是你写了一个手机APP来管理“To Do”list，你需要考虑哪些方面使它们变慢。

让我们来看下，reviewer在做code review时需要考虑的对性能有影响的一些事情。

### 1. 关于性能首先要考虑的事情
* 这些功能是否有硬性的性能要求？

  正在被review的这些代码以前有没有发布[SLA](https://en.wikipedia.org/wiki/Service-level_agreement)? 或者需求中有没有明确表述对性能的要求？

  如果原始的需求是伴随着“登录太慢”的一个bug, 原始的开发者应该明确合适的登录时间是多长 - 不然reviewer后者代码作者怎么能信心的保证登录速度已经提供到满意的程度了？

* 如果有，是否有测试已经证明性能要求已经满足？

    任何对性能要求很高的系统应该由自动化的性能测试来保证发布的SLAs("所有订单请求服务在10ms内响应")都满足了。没有这些自动化测试，你得依赖于你的用户来通知你没有达到SLAs的要求。这样不仅仅用户体验很差，而且会导致不可避免的罚款和费用。关于code review中的测试，请看[测试篇]。
* 新功能是否对任何已有的性能测试的结果有负面影响？

    如果你的性能测试是regularly运行的（或者你有一个测试集可以按需运行），用它们去检查一下在性能要求很要的区域的新代码没有引入任何使性能变差的东西。这个可能是一个自动化的处理过程，但是因为把性能测试放在CI环境中运行没有运行单元测试那么通用，这一点值得在code review中特别提及。
* 对code review是否有硬性的性能要求？

    花费时间放在那些痛苦的优化上来达到减少很少的CPU使用是非常低效的。但是reviewer可以通过检查来保证代码中没有那些常见的性能陷阱，这些完全可以通过code review去避免。

### 2. 请求外部的service/application的代价很高
对你自己应用只要的任何系统的调用都需要network hop，这个一般会比没有优化的equals(）方法的性能差的多。考虑一下几点：
* Database请求

  最坏的情况可能会隐藏在系统抽象的后面如ORMs。但是在做code review是你应该能够找到一些常见的会引起性能问题的原因，如在loop中的打个的database请求。看下面例子中的载入一组ID，然后通过这些ID逐个的请求database来获取对应结果。
  {% include image.html path="documentation/what-to-look-for-in-a-code-review/CR3-MultiDatabase.png" path-detail="documentation/what-to-look-for-in-a-code-review/CR3-MultiDatabase.png" alt="CR3-MultiDatabase" %}
  上面的列子中，19行上起来相当正常，但是它在一个‘for’循环中，你根本不知道这个代码会引起多少database请求。

* 不必要的网络请求

  和database一样， 远程service在很多时候会被过度使用，一个单一请求就能解决的问题用了多次远程调用，或者使用批处理或缓存能够阻止代价高昂的network请求。再次强调，和database一样，有事一个抽象可能掩盖一个实际上是远程API请求的方法调用。

* 移动、可穿戴Apps有过多的后台请求
  
  从本质上来说，这个和‘不必要的网络请求’是一样的，但是这个在移动设备上回带来额外的问题，不要的后台请求不但降低性能，而且会使电池寿命减短。

### 3. 有效并高效的使用资源
* 代码是否使用了锁来访问共享的资源？这样是否会导致低效或者死锁？
  
  [锁是性能杀手](https://mechanical-sympathy.blogspot.com.es/2013/08/lock-based-vs-lock-free-concurrent.html)并且在多线程环境中很难去理清楚。考虑这些模式：单线程去写或修改值，其他线程只读；或者使用[无锁算法](https://en.wikipedia.org/wiki/Non-blocking_algorithm).
* 代码是否存在导致内存泄露的可能？

  在Java中，一些常见的原因：可变静态字段，使用ThreadLocal和使用ClassLoader。

* 是否存在应用程序内存无限增长的可能？

  这个和内存泄露不太一样，内存泄露是一下不再被使用的对象没法被垃圾回收器回收。但是任何语言，甚至是没有垃圾回收机制的，都可以常见无限增长的数据结构。作为一个reviewer，如果你发现新的值被持续不断的加到一个list或者map中，你需要询问下这个list或者map什么时候失效或者清理。
  {% include image.html path="documentation/what-to-look-for-in-a-code-review/CR3-InfiniteMemory.png" path-detail="documentation/what-to-look-for-in-a-code-review/CR3-InfiniteMemory.png" alt="CR3-InfiniteMemory" %}

* 连接或流是否被关闭？

  关闭连接或者文件、网络流很容易被忘记。当你review其他人的代码是，不管是什么编程语言，要保证文件、网路、以及database连接被正确的关闭。
  {% include image.html path="documentation/what-to-look-for-in-a-code-review/CR3-CloseResources1.png" path-detail="documentation/what-to-look-for-in-a-code-review/CR3-CloseResources1.png" alt="CR3-CloseResources1" %}
  在Java 7中，很容易用[try-with-resource](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html)去解决这个问题。
  {% include image.html path="documentation/what-to-look-for-in-a-code-review/CR3-TryWithResources.png" path-detail="documentation/what-to-look-for-in-a-code-review/CR3-TryWithResources.png" alt="CR3-TryWithResources" %}

* 资源池是否配置正确？

  一个环境的最佳配置取决于多个因素，reviewer不太可能立即发现它们，如database连接池大小是否配置正确。但是这里有一些东西你一眼就能分辨出来，比如：pool太小（大小为1）或者太大（百万级线程）。想很好的调优这些值你得需要一个尽可能与实际产品环境接近的测试环境，但是有一个常见的问题可以再code review时候就能找出来，就是当一个pool（线程池或者连接池）太大。按道理说是越大越好，但是显而易见这些每一个对象都占用资源，管理成千上万个对象的成本一般要比它们带来的好处大的多。毫无疑问，把它们设在默认值一般是个不错的开始。如果这是设置偏离的默认设置，那么这些设置就需要测试或计算去证明为什么这个配置更好。

### 4. Reviewer可以很容易的找到警告信号⚠️
有些代码一眼就能找出其中潜在的性能问题，这些依赖于使用的编程语言和代码库。
* 反射

  在Java中，反射比没有反射要快。如果你在code review发现了它们，询问它们是不是必要的。
  {% include image.html path="documentation/what-to-look-for-in-a-code-review/CR3-Reflection.png" path-detail="documentation/what-to-look-for-in-a-code-review/CR3-Reflection.png" alt="CR3-Reflection" %}
  上面的方法是来自java.lang.reflect包，这里就需要注意一下了。

* 超时

  在review代码时候，你可能并不知道一个操作正确的超时是多久，但是你需要思考“当超时发生时对系统的其他部分影响是什么？”。作为reviewer，你需要考虑最差的情况-应用程序是否会就这么阻塞在那里当5mins超时时间到了后？当把超时时间设为1s时最坏的结果是什么？如果作者的代码不能调整超时时间长度，你作为reviewer不知道被选中的值的好处与坏处，这时你就要找到一个理解这个影响的人一起参与进来。千万不要等到你的用户告诉你关于性能的问题。

* 并行

  代码是否有用多线程去处理一个单一操作？这个是否会增加响应时间和复杂度而不是提高性能？在现代化的Java里面，相比较显示的创建一个线程，这个更难以发现。代码是否使用了Java8的并行流算法而没有从并行中得到好处？比如：在一个很少的元素组上使用并行流，或者在一个执行非常简单的操作的流上使用并行流，这样可能比在顺序的流上执行操作更慢。
  {% include image.html path="documentation/what-to-look-for-in-a-code-review/CR3-Parallel.png" path-detail="documentation/what-to-look-for-in-a-code-review/CR3-Parallel.png" alt="CR3-Parallel" %}

### 5. 正确性
这些不一定会影响你系统的性能，但是因为他们和运行多线程环境密切相关。
* 多线程环境下，代码是否使用了正确的数据结构？

   {% include image.html path="documentation/what-to-look-for-in-a-code-review/CR3-Threadsafe.png" path-detail="documentation/what-to-look-for-in-a-code-review/CR3-Threadsafe.png" alt="CR3-Threadsafe" %}
   上面的代码中，作者在12行用一个*ArrayList*去存所有的*sessions*。然而，这个代码，特别是*onOpen*方法，会被多线程调用，所以*sessions*字段需要一个线程安全的数据结构。在这个*case*里面，我们有一系列的选择：我们可以使用*Vector*，我们可以使用*Collection.synchronizedList()*去创建线程安全的*List*，但是这个*case*最好的选择应该是使用*CopyOnWriteArrayList，因为这个*list*的读操作远多于修改操作。
   {% include image.html path="documentation/what-to-look-for-in-a-code-review/CR3-Threadsafe2.png" path-detail="documentation/what-to-look-for-in-a-code-review/CR3-Threadsafe2.png" alt="CR3-Threadsafe2" %}
* 代码是否使用了竞争条件？

  在多线程环境里面，非常容易的写出引起不明显竞态条件的代码。比如：
  {% include image.html path="documentation/what-to-look-for-in-a-code-review/CR3-RaceCondition1.png" path-detail="documentation/what-to-look-for-in-a-code-review/CR3-RaceCondition1.png" alt="CR3-RaceCondition1" %}
  尽管这个用来增长的代码只有短短一行，但是很可能被其他线程调用来增加*orders*在这个代码取到值与设置新值这段时间里。注意*get*和*set*并不是原子性操作。

* 代码中的锁是否使用正确？

  和竞态条件相关，作为审查者你应该检查被审代码是否允许多个线程修改变量导致程序崩溃。代码可能需要同步、锁、原子变量来对代码块进行控制。

* 代码的性能测试是否是有价值的？

  很容易将小型的性能测试代码写得很糟糕，或者使用不能代表生产环境数据的测试数据，这样只会得到错误的结果。

* 缓存

  虽然缓存是一种能防止过多高消耗请求的方式，但其本身也存在一些挑战。如果审查的代码使用了缓存，你应该关注一些常见的问题，如，不正确的缓存失效方式。

### 6. 代码级的优化
对大部分并不是要构建低延时应用的机构来说，代码级优化往往是过早优化，所以首先要知道代码级优化是否必要。
* 代码是否在不需要的地方使用同步或锁操作？如果代码始终运行在单线程中，锁往往是不必要的。

* 代码是否可以使用原子变量替代锁或同步操作？

* 代码是否使用了不必要的线程安全的数据结构？比如是否可以使用ArrayList替代Vector？

* 代码是否在通用的操作中使用了低性能的数据结构？如在经常需要查找某个特定元素的地方使用链表。

* 代码是否可以使用懒加载并从中获得性能提升？

* 条件判断语句或其他逻辑是否可以将最高效的求值语句放在前面来使其他语句短路？

* 代码是否存在许多字符串格式化？是否有方法可以使之更高效？

* 日志语句是否使用了字符串格式化？是否先使用条件判断语句校验了日志等级，或使用延迟求值？
  {% include image.html path="documentation/what-to-look-for-in-a-code-review/CR3-Logging.png" path-detail="documentation/what-to-look-for-in-a-code-review/CR3-Logging.png" alt="CR3-Logging" %}
  上面的代码也在logger被设置为*FINEST*是才写log。但是，代价高昂的字符串格式化每次都会被调用，不管有没有信息被log。

  {% include image.html path="documentation/what-to-look-for-in-a-code-review/CR3-Logging2.png" path-detail="documentation/what-to-look-for-in-a-code-review/CR3-Logging2.png" alt="CR3-Logging2" %}
  按照上面这样的修改可以提高性能，只有在写log时才进行字符串格式化操作。

  {% include image.html path="documentation/what-to-look-for-in-a-code-review/CR3-Logging3.png" path-detail="documentation/what-to-look-for-in-a-code-review/CR3-Logging3.png" alt="CR3-Logging3" %}
  在Java 8里面，不用这个没用，但有不得不用的*if*也可以获得性能的提升，通过使用*lambda*表达式来做。因为使用*lambda*以后，字符串格式化操作也在真正有信息被log时才发生。这应该也是Java 8比较推荐的处理办法。

### 7. 关于性能有很多方面需要思考。。。
这里给出关于JVM优化性能的建议：
1. 写短小的类与方法；
2. 保持逻辑简单，不要有深层嵌套的*ifs*和*loops*
你的代码可读性越好，那么JIT compiler理解你的代码的机会越大，这样就更好做优化。这一点在code review过程中很容易找出来，即，我们闫让我们的代码可理解性好，要整洁，这样的代码的性能一般表现得都比较好。

### 8. 写在最后
在做关于性能这块code review时，切记你能够在某些地方很快的发现性能问题（如，不必要的database请求），在某些地方可以有一些尝试的做法（如，代码级的优化）来获得性能的提升。











