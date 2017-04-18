---
layout: post
title: "服务器端应用安全"
description: "常见的服务器端应用安全问题描述"
tags: [web, security]
---
注：原文来自《白帽子讲Web安全》

* 目录
{:toc}

## 1. 注入攻击
注入攻击是Web 安全领域中一种最为常见的攻击方式。XSS 本质上也是一种针对HTML 的注入攻击。`数据与代码分离原则`可以说是专门为了解决注入攻击而生的。

注入攻击的本质，是把用户输入的数据当做代码执行。这里有`两个关键条件`：**第一个是用户能够控制输入；第二个是原本程序要执行的代码，拼接了用户输入的数据。**

### SQL 注入
在今天，SQL 注入对开发者来说，应该是耳熟能详了。而SQL 注入第一次为公众所知，是在1998年的著名黑客杂志《Phrack》第54期上，一位名叫rfp的黑客发表了一篇题为[”NT Web Technology Vulnerability“](http://phrack.org/issues/54/8.html)的文章。

在文章中，第一次向公众介绍了这种新型的攻击技巧。下面是一个SQL 注入的典型例子。
{% highlight javascript %}
var Shipcity;
Shipcity =  Request.form("Shipcity");
var sql = "select * from OrdersTable where Shipcity = '" + Shipcity + "'";
{% endhighlight %}

变量Shipcity 的值由用户提交，在正常情况下，假如用户输入”Beijing“,那么SQL 语句会执行：
{% highlight SQL %}
select * from OrdersTable where Shipcity = 'Beijing'
{% endhighlight %}

但假如用户输入一段有语义的SQL语句，比如：
{% highlight javascript %}
Beijing'; drop table OrdersTable--
{% endhighlight %}

那么，SQL 语句的实际执行就会如下：
{% highlight SQL %}
select * from OrdersTable where Shipcity = 'Beijing'; drop table OrdersTable--'
{% endhighlight %}

我们看到，原本正常执行的查询语句，现在变成了查询完成后，再执行一个drop 表的操作，而这个操作，是用户构造了恶意数据的结果。

回过头来看看注入攻击的两个条件：
* 用户能够控制数据的输入 -- 在这里，用户能够控制变量 Shipcity。
* 原本要执行的代码，拼接了用户的输入：
{% highlight javascript %}
var sql = "select * from OrdersTable where Shipcity = '" + Shipcity + "'";
{% endhighlight %}
这个”拼接“的过程很重要，正是这个拼接的过程导致了代码的注入。

在SQL注入的过程中，如果网站的Web 服务器开启了错误回显，则会为攻击者提供极大的便利，比如攻击者在参数中输入一个单引号”'“，引起执行查询语句的语法错误，服务器直接返回了错误信息：
{% highlight javascript %}
Microsoft JET Database Engine 错误 ’80040e14‘
字符串的语法错误，在查询表达式 ’ID = 49‘’ 中。
/showdetail.asp， 行8
{% endhighlight %}

从错误信息可以知道，服务器用的是Access 作为数据库，查询语句的伪代码极有可能是
{% highlight SQL %}
select xxx from table_X where id = $id
{% endhighlight %}
错误回显披露了敏感信息，对于攻击者来说，构造SQL注入的语句就可以更加得心应手了。

### XML 注入
XML 是一种常见的标记语言，通过标签对数据进行结构化表示。XML 与HTML 都是SGML（Standard Generalized Markup Language，标准通用标记语言）。

XML 与HTML 一样，也存在注入攻击，甚至在注入的方法上也非常相似。如下例子，这段代码将生成一个XML 文件。
{% highlight javascript %}
final String GUESTROLE = "guest_role";
...
//userdata是准备保存的XML数据，接收了name和email两个用户提交来的数据
String userdata = "<USER role=" + 
    GUESTROLE + 
    "><name>" + 
    request.getParameter("name") +
    "</name><email>" +
    request.getParameter("email") + 
    "</email></USER>";
//保存XML数据
userDao.save(userdata);
{% endhighlight %}

但是如果用户构造了恶意输入数据，就有可能形成注入攻击。比如用户输入的数据如下
{% highlight xml %}
user1@a.com</email></USER>
<USER role="admin_role">
    <name>test</name>
    <email>user2@a.com
{% endhighlight %}

最终生成的XML文件里被插入一条数据：
{% highlight xml %}
<USER role="guest_role">
    <name>user1</name>
    <email>user1@a.com</email>
</USER>
<USER role="admin_role">
    <name>test</name>
    <email>user2@a.com</email>
</USER>
{% endhighlight %}

XML 注入，也需要满足注入攻击的两大条件：用户能控制数据的输入；程序拼凑了数据。

### 代码注入
代码注入比较特别一点。代码注入与命令注入往往都是由一些不安全的函数或者方法引起的，其中的典型代表就是eval()。如下例：
{% highlight php %}
$myvar = "varname";
$x = $_GET['arg'];
eval("\$myvar = $x;");
{% endhighlight %}

攻击者可以通过如下Payload 实施代码注入：
{% highlight php %}
/index.php?arg=1; phpinfo()
{% endhighlight %}

存在代码注入漏洞的地方，与”后门“没有区别。

在Java中也可以实施代码注入，比如利用Java的脚本引擎。
{% highlight java %}
import javax.script.*;

public class Example{
  public static void main(String[] args){
    try{
      ScriptEngineManager manager = new ScriptEngineManager();
      ScriptEngine engine = manager.getEngineByName("JavaScript");
      System.out.println(args[0]);
      engine.eval("print('" + args[0] + "')");
    }catch(Exception e){
      e.printStackTrace(); 
    }    
  }
}
{% endhighlight %}

攻击者可以提交如下数据：
{% highlight javascript %}
hallo'); var fImport = new JavaImporter(java.io.File);
with(fImport){var f =  new File('name'); f.createNewFile();} //
{% endhighlight %}

此外，JSP 的动态include 也能导致代码注入。严格来说，PHP、JSP的动态include（文件包含漏洞）导致的代码执行，都可以算是一种代码注入。
{% highlight jsp %}
<% String pageToInclude = getDataFromUntrustedSource(); %>
<jsp:include page="<%=pageToInclude %>" />
{% endhighlight %}
代码注入多见于脚本语言，有时候代码注入可以造成命令注入（Command Injection）。

代码注入往往是由于不安全的编程习惯所造成的，危险函数应该尽量避免在开发中使用，可以在开发规范中明确指出哪些函数是禁止使用的。这些危险函数一般在开发语言的官方文档中可以找到一些建议。

### CRLF 注入
CRLF 实际上是两个字符：CR 是Carriage Return（ASCII 13，\r)，LF 是Line Feed（ASCII 10，\n）。\r\n这两个字符用于表示换行的，其十六进制编码分别为 Ox0d、Ox0a。

CRLF 常被用作不同语义之间的分隔符。因此通过”注入CRLF 字符“，就有可能改变原有的语义。

比如，在日志文件中，通过CRLF 有可能构造出一条心得日志。下面这段代码，将登陆失败用户名写入日志文件中。
{% highlight python %}
def log_failed_login(username)
  log = open("access.log", 'a')
  log.write("User login failed for: %s\n" % username)
  log.cloase
{% endhighlight %}

在正常情况下，会记录下如下日志：
{% highlight python %}
User login failed for: guest
{% endhighlight %}

但是由于没有处理换行符"\r\n"，因此当攻击者输入如下数据时，就可能插入一条额外的日志记录。
{% highlight python %}
guest\nUser login failed for: admin
{% endhighlight %}

日志文件因为换行符”\n“的存在，会变成：
{% highlight python %}
User login failed for: guest
User login failed for: admin
{% endhighlight %}

第二条记录是伪造的，admin用户并不曾登陆失败。

CRLF 注入并非仅能用于log注入，凡是使用CRLF作为分隔符的地方都可能粗在这种注入，比如”注入HTTP头“。

在HTTP协议中，HTTP头是通过"\r\n"来分隔的。因此如果服务器端没有过滤"\r\n"，而又把用户输入的数据放在HTTP头中，则有可能导致安全隐患。这种在HTTP头中的CRLF注入，又称为**”Http Response Splitting“。

”HTTP Response Splitting“的危害甚至比XSS还要大，因为它破坏了HTTP协议的完整性。

对抗CRLF 的方法非常简单，只需要处理好"\r"、"\n"这两个保留字即可，尤其是那些使用”换行符“作为分隔符的应用。

## 2. 文件上传漏洞
**文件上传漏洞是指用户上传了一个可执行的脚本文件，并通过此脚本文件获得了执行服务器端命令的能力。**这种攻击方式是最为直接和有效的，有时候几乎没有什么技术门槛。

在互联网中，我们经常用到文件上传功能，比如上传一张自定义的图片：分享一段视频或者照片；论坛发帖时附带一个附件；在发送邮件时附带附件，等等。

文件上传功能本身是一个正常业务需求，对于网站来说，很多时候也确实需要用户将文件上传到服务器。`所以”文件上传“本身没有问题，但有问题的是文件上传后，服务器怎么处理，解释文件`。如果服务器的处理逻辑做的不够安全，则会导致严重的后果。

文件上传后导致的常见问题一般有：
* 上传文件是Web脚本语言，服务器的Web容器解释并执行了用户上传的脚本，导致代码执行；
* 上传文件是Flash的策略文件crossdomain.xml，黑客用以控制Flash在该域下的行为（其他通过类似方式控制策略文件的情况类似）；
* 上传文件是病毒、木马文件，黑客用以诱骗用户或者管理员下载执行；
* 上传文件是钓鱼图片或者包含了脚本的图片，在某些版本的浏览器中会被作为脚本执行，被用于钓鱼和欺诈。

除此之外，还有一些不常见的利用方法，比如将文件上传作为一个入口，溢出服务器的后台处理程序，如图片解析模块；或者上传一个合法的文本文件，其内容包含了PHP脚本，在通过”本地文件包含漏洞（Local File Include）“执行此脚本；等等。

在大多数情况下，文件上传漏洞一般都是指”上传Web脚本能够被服务器解析“的问题，也就是通常所说的webshell的问题。**要完成这个攻击，要满足如下几个条件**：
* *首先*，上传的文件能够被Web 容器解析执行。所以文件上传后所在的目录要是Web容器所覆盖到的路径。
* *其次*，用户能够从Web上访问这个文件。如果文件上传了，但用户无法通过Web访问，或者无法使得Web容器解析这个脚本，那么也不能称之为漏洞。
* *最后*，用户上传的文件若被安全检查、格式化、图片压缩等功能改变了内容，则也可能导致攻击不成功。

## 3. 认证与会话管理
”认证“是最容易理解的一种安全。最常见的认证方式就是用户名与密码，但认证的手段却远远不止于此。

### Who am I ?
很多时候，人们会把”认证“和”授权“两个概念搞混。区分这两个概念，只需记住下面这个事实：

**认证的目的是为了认出用户是谁，而授权的目的是为了决定用户能做什么。**

认证的手段是多样化的，其目的就是为了能够识别出正确的人。如何才能准确的判断一个人是谁呢？这是一个哲学问题，在被哲学家们搞清楚之前，我们只能依据人的不同”凭证“来确定一个人的身份。钥匙仅仅是一个很脆弱的凭证，其他诸如指纹、虹膜、人脸、声音等生物特征也是能够识别一个人的凭证。`认证实际上就是一个验证凭证的过程。`

如果只有一个凭证被用于认证，则称为”单因素认证“；如果有两个或多个凭证被用于认证，则称为”双因素认证“或”多因素认证“。

### Session 与认证
密码与证书等认证手段，一般仅仅用于登陆的过程。当登陆完成后，用户访问网站的页面，不可能每次浏览器请求页面时都再使用密码认证一次。因此，当认证成功后，就需要替换一个对用户透明的凭证。这个凭证就是`SessionID`。

当用户登陆完成后，在服务器端就会创建一个新的会话（Session），会话中会保存用户的状态和相关信息。服务器端维护所有在线用户的Session，此时的认证，只需要知道哪个用户在浏览当前的页面即可。为了告诉服务器应该使用哪一个Session，浏览器需要把当前用户持有的SessionID告知服务器。

最常见的做法就是把SessionID 加密后保存在Cookie中，因为Cookie会随着HTTP请求头发送，且受到浏览器同源策略的保护。

SessionID一旦在生命周期内被窃取，就等同于账户失窃。同时由于SessionID是用户等咯之后才持有的认证凭证，因此黑客不需要再攻击登陆过程（比如密码），在设计安全方案时需要意识到这一点。

Session劫持就是一种通过窃取用户SessionID后，使用该SessionID登陆进目标账户的攻击方法，此时攻击者实际上是使用了目标账户的有效Session。如果SessionID是保存在Cookie中的，则这种攻击可以称为Cookie劫持。

Cookie泄露的途径有很多，最常见的有XSS攻击、网络Sniff、以及本地木马窃取。对于通过XSS漏洞窃取Cookie的攻击，通过给Cookie标记HTTPonly，可以有效地缓解XSS窃取Cookie的问题。但是其他的泄露途径，比如网络被嗅探，或者Cookie文件被窃取，则会涉及客户端的环境安全呢，需要从客户端着手解决。

## 4. 访问控制
”权限“一词在安全领域出现的频率很高。”权限“实际上是一种”能力“。对于权限的合理分配，一直是安全设计中的核心问题。在互联网安全领域，尤其是Web安全领域中，”权限控制“的问题都可以归结为”访问控制“的问题，这种描述也更精准一些。

### What Can I Do?
权限控制，或者说访问控制，广泛应用于各个系统中。抽象地说，`都是某个主体（subject）对某个客体（object）需要实施某种操作（operation），而系统对这种操作的限制就是权限控制`。

在网络中，为了保护网络资源的安全，一般是透过路由设备或者防火墙建立基于IP 的访问控制。这种访问控制的”主体“是网络请求的发起方（比如一台PC），”客体“是网络请求的接收方（比如一台服务器），主体对客体的”操作“是对客体的某个端口发起网络请求。这个操作能否执行成功，是受到防火墙ACL策略限制的。

在操作系统中，对文件的访问也有访问控制。此时”主体“是系统的用户，”客体“是被访问的文件，能否访问成功，将由操作系统给文件设置的ACL（访问控制列表）决定。比如在Linux系统中，一个文件可以执行的操作分为”读“、”写“、”执行“三种，分别由r\w\x表示。这三种操作同时对应着三种主体：文件拥有者、文件拥有者所在的用户组、其他用户。主体、客体、操作者三者之间的对应关系，构成了访问控制列表。

在一个安全系统中，确定主体的身份是”认证“解决的问题；而客体是一种资源，是主体发起的请求的对象。**在主体对客体进行操作的过程中，系统控制主体不能”无限制“地对客体进行操作，这个过程就是”访问控制“。**

主体”能够做什么“，就是权限。权限可以细分成不同的能力（capability）。

在Web应用中，根据访问客体的不同，常见的访问控制可以分为”基于URL的访问控制“、”基于方法（method）的访问控制“和”基于数据的访问控制“。

### OAuth
OAuth 是一个在不提供用户名和密码的情况下，授权第三方应用访问Web 资源的安全协议。OAuth 1.0 于2007年12月公布，并迅速成为了行业标准。2010年4月，OAuth 1.0 正式成为了RFC 5849。

OAuth 与 OpenID 都致力于让互联网变得更加开放。**OpenID 解决的是认证问题，OAuth 则更注重授权。** 认证与授权的关系其实是一脉相承的，后来人们发现，其实更多的时候真正需要的是对资源的授权。

OAuth 委员会实际上是从OpenID 委员会中分离出来的（2006年12月），OAuth 的设计原本想弥补OpenID 中的一些缺陷或者说不够方便的地方，但后来发现需要设计一个全新的协议。

常见的应用OAuth 的场景，一般是某个网站想要获取一个用户在第三方网站中的某些资源服务。比如在简书上，想要导入微博里的头像与名称。（例子来源[简书](http://www.jianshu.com/p/0db71eb445c8)）

**原理概要**

新浪微博作为服务提供商，拥有用户的头像、昵称、邮箱、好友以及所有的微博内容，简书希望获取用户存储在微博的头像和昵称，假设它们是三个人：

1. 简书问新浪微博：我想要获取用户 A 的头像和昵称，请你提供
2. 微博说：我需要经过用户Ａ 本人的许可，然后去问用户 A 是否要授权简书访问自己的头像和昵称
3. 用户 A 对微博说：我给简书一个临时的钥匙，如果他给你出示了这把钥匙，你就把我的资料给他
4. 简书使用户给它的钥匙获取用户头像和昵称信息。

以上是 OAuth 认证的大概流程。在使用微博授权之前，简书需要先在微博开放平台上注册应用，填写自己的名称、logo、用途等信息，微博开放平台颁发给简书一个应用 ID 和叫 APP Secret 的密钥，在实际对接中，会使用到这两个参数。

**详细流程**
{% include image.html path="documentation/white-hat-web-security/oauth-intro.jpeg" path-detail="documentation/white-hat-web-security/oauth-intro.jpeg" alt="oauth-intro" %}

## 5. 加密算法与随机数
加密算法与伪随机数算法是开发中经常会用到的东西，但是加密算法的专业性非常强，在Web开发中，如果对加密算法和伪随机数算法缺乏一定的了解，则很有可能会错误地使用它们，最终导致应用出现安全问题。

### 加密算法概述
密码学有着悠久的历史，它满足了人们对安全的最基本需求 -- 保密性。密码学可以说是安全领域发展的基本。
<div align='center'>
{% include image.html path="documentation/white-hat-web-security/davinci-code-bucket.jpg" path-detail="documentation/white-hat-web-security/davinci-code-bucket.jpg" alt="davinci-code-bucket" %}
</div>

在Web应用中，最常见的就是网站在将敏感信息保存到Cookie时使用的加密算法。加密算法的运用是否正确，与网站的安全息息相关。

常见的加密算法通常为**分组加密算法**和**流加密算法**两种，两者的实现原理不同。

分组加密算法基于”分组“（block）进行操作，根据算法的不同，每个分组的长度可能不同。分组加密算法的代表有DES、3-DES、Blowfish、IDEA、AES等。下图演示了一个使用CBC模式的分组加密算法的加密过程。
{% include image.html path="documentation/white-hat-web-security/Cbc_encryption.png" path-detail="documentation/white-hat-web-security/Cbc_encryption.png" alt="Cbc_encryption" %}

流加密算法，则每次只处理一个字节，密钥独立于消息之外，两者通过异或实现加密与解密。流加密算法的代表有RC4、ORYX、SEAL等。下图演示了流加密算法的加密过程。流加密算法
<div align='center'>
{% include image.html path="documentation/white-hat-web-security/stream-cipher.jpeg" path-detail="documentation/white-hat-web-security/stream-cipher.jpeg" alt="stream-cipher" %}
</div>

针对加密算法的攻击，一般根据攻击者能获得的信息，可以分为：
* 唯密文攻击：攻击者有一些密文，它们是使用同一加密算法和同一密钥加密。这种攻击最难。
* 已知明文攻击：攻击者除了能得到一些密文外，还能得到这些密文对应的明文。
* 选择明文攻击：攻击者不仅能得到一些密文和明文，还能选择用于加密的明文。
* 选择密文攻击：攻击者可以选择不同的密文来解密。

### 密钥管理
在密码学里有个基本的原则：**密码系统的安全应该依赖于密钥的复杂性，而不应该依赖于算法的保密性。**

在安全领域里，选择一个足够安全的加密算法不是困难的事情，难的是密钥管理。在一些实际的攻击案例中，直接攻击加密算法本身的案例很少，而因为密钥没有妥善管理导致的安全事件却很多。对于攻击者来说，他们不需要正面破解加密算法，如果能够通过一些方法获得密钥，则是件事半功倍的事情。

**密钥管理中最常见的错误，就是将密钥硬编码在代码里**。同样的，将加密密钥、签名的salt等”key“硬编码在代码中，是非常不好打额习惯。

对于Web应用来说，常见的做法是将密钥（包括密码）保存在配置文件或者数据库中。在使用时由程序读出密钥并加载进内存。密钥所在的配置文件或数据库需要严格的控制访问权限，同时也要确保运维或DBA中具有访问权限的人越少越好。

在应用发布到生成环境时，需要重新生成新的密钥或密码，以免与测试环境中使用的密钥相同。

当黑客已经入侵后，密钥管理系统也难以保证密钥的安全性。比如攻击者获取了一个webshell，那么攻击者也就具备了应用程序的一切权限。由于正常的应用程序也需要使用密钥，因此对密钥的控制不可能限制住Webshell的”正常“请求。

密钥管理的主要目的，还是为了`防止密钥从非正常的渠道泄露`。定期更换密钥也是一种有效地做法。一个比较安全的密钥管理系统，可以将所有的密钥（包括一些敏感配置文件）都几种保存在一个服务器（集群）上，并通过Web Service的方式提供获取密钥的API。每个Web应用在需要使用密钥时，通过带认证信息的API请求密钥管理系统，动态获取密钥。Web应用不能把密钥写入本地文件中，只加载到内存，这样动态获取密钥最大程度地保护了密钥的私密性。密钥几种管理，降低了系统对于密钥的耦合性，也有利于定期更换密钥。

### 伪随机数的问题
伪随机数（pseudo random number）的问题 -- 伪随机数`不够随机`，是程序开发中会出现的一个问题。一方面，大多数开发者对此方面的安全知识有所欠缺，很容易写出不安全的代码；另一方面，伪随机数的问题的攻击方式在多数情况下都只存在于理论中，难以证明，因此在说服程序员修补代码时也显得有点理由不够充分。

但伪随机数的问题是真实存在的、不可忽略的一个安全问题。伪随机数，是通过一些数学算法生成的随机数，并非真正的随机数。密码学上的安全伪随机数应该是不可压缩的。对应的”真随机数“，则是通过一些物理系统生成的随机数，比如电压的波动、硬盘磁头读/写时的寻道空间、空中电磁波的噪音等。

### 最佳实践
在加密算法的选择和使用上，有以下最佳实践：
* 不使用ECB（Electronic codebook）模式；
* 不要使用流密码（如RC4）；
* 使用HMAC-SHA1代替MD5(甚至是代替SHA1)；
* 不要使用相同的key做不同的事情；
* salts与IV需要随机产生；
* 不要自己实现加密算法，尽量使用安全专家已经实现好的库；
* 不要依赖系统的保密性。

当你不知道该如何选择时，有以下建议：
* 使用CBC（Cipher-Block-Chaining）模式的AES256 用于加密；
* 使用HAMC-SHA512用于完整性检查；
* 使用带salt的SHA-256或SHA-512用于Hashing。

## 6. Web 框架安全
在实施安全时，要达到好的效果，必要要完成两个目标，（1）安全方案正确、可靠；（2）能够发现所有可能存在的安全问题，不出现遗留。

只有深入理解漏洞原理之后，才能设计出真正有效、能够解决问题的方案。但是，方案光有效还是不够的，要想设计出完美的方案，还需要解决第二件事情，就是找到一个方法，能够让我们快速有效、不会遗漏地发现所有问题。而Web 开发框架，为我们解决了这个问题提供了便捷。

在框架中实施安全方案，比由程序员在业务中修复一个个具体的bug,有着更多的优势。
* 首先，有些安全问题可以在框架中统一解决，能够大大节省程序员的工作量，节约人力成本。当代码的规模大道一定程度时，在业务的压力下，专门花时间去一个个修补漏洞几乎成为不可能完成的任务。
* 其次，对于一些常见的漏洞来说，由程序员一个个修补可能会发现遗漏，而在框架中统一解决，有可能解决”遗漏“问题。这需要制定相关的代码规范和工具配合。
* 最后，在每个业务里修补安全漏洞，补丁的标准难以统一，而在框架中集中实施的安全方案，可以使所有基于框架开发的业务都能受益，从安全方案的有效性来说，更容易把握。

**同时Web框架自身的安全同样不可忽视，作为一个基础服务，一旦出现漏洞，影响是巨大的。**

## 7. 应用层拒绝服务攻击
在互联网中一谈起DDOS攻击，人们往往谈虎色变。DDOS攻击被认为是安全领域中最难解决的问题之一，迄今为止也没有一个完美的解决方案。

DDOS 又称为分布式拒绝服务，全称是Distributed Denial Of Service。DDOS 本是利用合理的请求造成资源过载，导致服务不可用。比如一个停车场共有100个车位，当100个车位都停满车后，再有车想要停进来，就必须等已有的车先出去才行。如果已有的车一直不出去，那么停车场的入口就会排起长队。停车场的负荷过载，不能正常工作了，这种情况就是“拒绝服务”。

分布式拒绝服务攻击，将正常请求放大了若干倍，通过若干个网络节点同时发起攻击，已达成规模效应。这些网络节点往往是黑客们所控制的“肉鸡”，数量达到一定规模后，就形成了一个“僵尸网络”。大型的僵尸网络，甚至达到了数万、数十万台的规模。如此规模的僵尸网络发起的DDOS攻击，几乎是不可阻挡的。

常见的DDOS 攻击有SYN flood、UDP flood、 ICMP flood等。

## 8. 编程语言的安全
现在各种编程语言都有Secure Code Guidelines，我们可以在对应的官网上找到。例如：

[Java SE Secure Code Guidelines](http://www.oracle.com/technetwork/java/seccodeguide-139067.html
)

## 9. Web Server 配置安全
Web 服务器是Web 应用的载体，Web 服务器安全，考虑的是应用部署时的运行环境安全。这个运行环境包括Web Server、脚本语言解释器、中间件等软件，这些软件所提供的一些配置参数，也可以起到安全保护的作用。

Web服务器常见的运行时安全有：
* Apache 安全
* Nginx 安全
* jBoss 远程命令执行
* Tomcat 远程命令执行
* HTTP Parameter Pollution