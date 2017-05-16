---
layout: post
title: "Web应用程序渗透测试方法论 - 测试特殊功能方面的输入漏洞"
description: "应用程序中有一系列漏洞只有在特殊功能中才会表现出来"
tags: [web, security, test]
---
注：原文来自《黑客攻防技术宝典》

* 目录
{:toc}

{% include image.html path="documentation/pen-test/test-func-spec-input-vulners.png" path-detail="documentation/pen-test/test-func-spec-input-vulners.png" alt="test-func-spec-input-vulners" %}

## 1. 测试SMTP注入
(1) 对于与电子邮件有关的功能使用的每一个请求，轮流提交以下每个测试字符串作为每一个参数，并在相关位置插入电子邮件地址。可以使用Burp Intruder自动完成这项任务。
{% highlight markdown %}
<youremail>%0aCc:<youremail>

<youremail>%0d%0aCc:<youremail>

<youremail>%0aBcc:<youremail>

<youremail>%0d%0aBcc:<youremail>

%0aDATA%0afoo%0a%2e%0aMAIL+FROM:+<youremail>%0aRCPT+TO:+<youremail>

%0aDATA%0aFrom:+<youremail>%0aTo:+<youremail>%0aSubject:+test%0afoo
%0a%2e%0a
%0d%0aDATA%0d%0afoo%0d%0a%2e%0d%0aMAIL+FROM:+<youremail>%0d%0aRCPT
+TO:+
<youremail>%0d%0aDATA%0d%0aFrom:+<youremail>
{% endhighlight %}

(2) 检查测试结果，确定应用程序返回的所有错误消息。如果这些错误与电子邮件功能中的问题有关，确定是否有必要调整输入，以利用漏洞。

(3) 应该监控指定的电子邮件地址，看是否收到任何电子邮件。

(4) 仔细检查生成相关请求的HTML表单。它们可能提供与服务器软件有关的线索。其中可能包含一个用于指定电子邮件收信人（To）地址的隐藏或禁用字段，可以直接对其进行修改。

## 2. 测试本地代码漏洞
### 2.1 测试缓冲区溢出
(1) 向每一个目标数据提交一系列稍大于常用缓冲区大小的长字符串。一次针对一个数据实施攻击，最大程度地覆盖应用程序中的所有代码路径。可以使用Burp Intruder中的字符块有效载荷来源自动生成各种大小的有效载荷。可以对下面的缓冲区大小进行测试：
{% highlight markdown %}
1100
4200
33000
{% endhighlight %}

(2) 监控应用程序的响应，确定所有反常现象。任何无法控制的溢出几乎可以肯定会在应用程序中造成异常，虽然远程诊断问题的本质可能非常困难。寻找一下反常现象。
* HTTP 500状态码或错误消息，这时其他畸形（而非超长）输入不会产生相同的结果。
* 内容详细的消息，表示某个外部本地代码组件发生故障。
* 服务器受到一个局部或畸形响应。
* 服务器的TCP连接未返回响应，突然关闭。
* 整个Web应用程序停止响应。
* 应用程序返回出人意料的数据，表示内存中的一个字符串可能“丢失”了它的空终止符。

### 2.2 测试整数漏洞
(1) 当测试本地代码组件时，确定所有基于整数的数据，特别是长度指示符，可以利用它触发整数漏洞。

(2) 向每一个目标数据提示旨在触发漏洞的适当有效载荷。轮流向每一个目标数据发送一系列不同的值，分别表示不同大小的有符号与无符号整数值的边界情况，如下所示。
* 0x7f and 0x80 (127 and 128)
* 0xff and 0x100 (255 and 256)
* 0x7ffff and 0x8000 (32767 and 32768)
* 0xffff and 0x10000 (65535 and 65536)
* 0x7fffffff and 0x80000000 (2147483647 and 2147483648)
* 0xffffffff and 0x0 (4294967295 and 0)

(3) 当被修改的数据以十六进制表示时，应该发送每个测试字符串的little-endian与big-endian版本，例如，ff7f以及7fff。如果十六进制数字以ASCII形式提交，应该使用应用程序自身使用的字母字符，确保这些字符被正确编码。

(4) 监控应用程序的响应，查找所有反常事件。

### 2.3 测试格式化字符串漏洞
(1) 轮流向每一个参数提交包含一长串不同格式说明符的字符串。例如：
{% highlight markdown %}
%n%n%n%n%n%n%n%n%n%n%n%n%n%n%n%n%n%n%n%n
%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s
%1!n!%2!n!%3!n!%4!n!%5!n!%6!n!%7!n!%8!n!%9!n!%10!n! etc...
%1!s!%2!s!%3!s!%4!s!%5!s!%6!s!%7!s!%8!s!%9!s!%10!s! etc...
{% endhighlight %}
`记得将%字符进行URL编码。`

(2) 监控应用程序的响应，查找所有反常事件。

## 3. 测试SOAP注入
(1) 轮流测试怀疑通过SOAP消息处理的参数。提交一个恶意XML结束标签，如</foo>。没有发生错误就表示该输入可能没有插入SOAP消息中，或者以某种方式被净化。

(2) 出现错误就提交一对有效地起始与结束标签，如<foo></foo>。如果这对标签使错误消失，应用程序很可能易于受到攻击。

(3) 如果提交的攻击字符串在应用程序的响应中原样返回，轮流提交下面两个值。如果发现其中一个值的返回结果为另一个值，或者只是返回test，那么可以确信该输入被插入到了XML消息中。
{% highlight markdown %}
test<foo/>
test<foo></foo>
{% endhighlight %}

(4) 如果HTTP请求中包含几个可放入SOAP消息中的参数，尝试在一个参数中插入起始注释字符&#x3C;!--， 在另一个参数中插入结束注释字符!--&#x3E;。 然后，轮换在参数中插入这两个字符（因为无法知道参数出现的顺序）。这样做可能会把服务器SOAP消息的某个部分作为注释处理，从而改变应用程序的逻辑，或者形成一个可能造成信息泄露的错误条件。

## 4. 测试LDAP注入
(1) 在任何使用用户提交的数据从一个目录服务中获取信息的功能中，针对每一个参数，轮流测试是否可以注入LDAP查询。

(2) 提交*字符。返回大量结果就明确表示针对的是LDAP查询。

(3) 尝试输入大量闭括号：
{% highlight markdown %}
))))))))))
{% endhighlight %}

(4) 尝试输入干扰不同查询的各种表达式，看是否影响返回的结果。在查询目录未知的情况下`cn`非常有用，因为所有的LDAP实现都支持这个特性。
{% highlight markdown %}
)(cn=*
*))(|(cn=*
*))%00
{% endhighlight %}

(5) 尝试在输入结尾增加其他属性，并用逗号分隔这些属性。轮流测试每一个属性，返回错误消息就表示该属性当前无效。以下属性常用在有LDAP查询的目录中：
{% highlight markdown %}
cn
c
mail
givenname
o
ou
dc
l
uid
objectclass
postaladdress
dn
sn
{% endhighlight %}

## 5. 测试XPATH注入
(1) 尝试提交下面的值，并确定它们是否会使用应用程序的行为发生改变，但不会造成错误：
{% highlight markdown %}
‘ or count(parent::*[position()=1])=0 or ‘a’=’b
‘ or count(parent::*[position()=1])>0 or ‘a’=’b
{% endhighlight %}

(2) 如果参数为数字，尝试提交下面的测试字符串：
{% highlight markdown %}
1 or count(parent::*[position()=1])=0
1 or count(parent::*[position()=1])>0
{% endhighlight %}

(3) 如果上面的任何字符串导致应用程序的行为发生改变，但不会造成错误，很可能可以通过设计测试条件，一次提取一字节的信息，从而获取任意数据。使用一系列以下格式的条件确定当前节点的父节点的名称：
{% highlight markdown %}
substring(name(parent::*[position()=1]),1,1)=’a’
{% endhighlight %}

(4) 提取出父节点的名称后，使用一系列下列格式的条件提取XML树中的所有数据。
{% highlight markdown %}
ssubstring(//parentnodename[position()=1]/child::node()[position()=1]
/text(),1,1)=’a’
{% endhighlight %}

## 6. 测试后端请求注入
(1) 确定在参数中指定内部服务器名称或IP地址的任何情况。提交任意服务器和端口，监视应用程序是否出现超时。还可以提交localhost，然后提交自己的IP地址，之后在指定端口上监视传入连接。

(2) 针对根据特定值返回特定页面的请求参数，尝试使用以下各种语法附加新的注入参数：
{% highlight markdown %}
%26foo%3dbar (URL-encoded &foo=bar)
%3bfoo%3dbar (URL-encoded ;foo=bar)
%2526foo%253dbar (Double URL-encoded &foo=bar)
{% endhighlight %}
如果应用程序的行为与未修改原始参数时相同，说明其中可能存在HTTP参数注入漏洞。这时，可通过注入可能更改后端逻辑的已知参数的名/值对来攻击后端请求。（参考书第10章）

## 7. 测试XXE注入
如果用户正向服务器提交XML，则可以实施外部实体注入攻击。如果已知会向用户返回某个字段，可以尝试制定一个外部实体，如下所示：
{% highlight markdown %}
POST /search/128/AjaxSearch.ashx HTTP/1.1
Host: mdsec.net
Content-Type: text/xml; charset=UTF-8
Content-Length: 115
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM “file:///windows/win.ini” > ]>
<Search><SearchTerm>&xxe;</SearchTerm></Search>
{% endhighlight %}
如果找不到已知字段，可以将“http://192.168.1.1:25”指定为外部实体，并监视页面响应时间。如果页面响应的时间明显增长或页面超时，则说明应用程序易于受到攻击。

