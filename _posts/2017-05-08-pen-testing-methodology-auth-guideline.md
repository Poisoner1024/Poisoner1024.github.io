---
layout: post
title: "Web应用程序渗透测试方法论 - 测试验证机制"
description: "验证机制是客户端向服务器证明其身份的特殊方法"
tags: [web, security, test]
---
注：原文来自《黑客攻防技术宝典》

* 目录
{:toc}

{% include image.html path="documentation/pen-test/test-auth-mech.png" path-detail="documentation/pen-test/test-auth-mech.png" alt="test-auth-mech" %}

## 1. 了解验证机制
(1) 确定应用程序使用的验证技术（如表单、证书或多元机制）。

(2) 确定所有与验证有关的功能（如登陆、注册、账户恢复等）。

(3) 如果应用程序并未采用自动自我注册机制，确定是否可以使用任何其他方法获得几个用户账户。

## 2. 测试密码强度
(1) 在应用程序中查找有关用户密码最小强度规则的说明。

(2) 尝试使用所有自我注册或密码修改功能，设定各种脆弱密码，确定应用程序实际应用的密码强度规则。尝试使用短密码、仅包含字母字符的密码、全部大写或全部小写字符的密码、单词型密码以及将当前用户名作为密码。

(3) 测试不完整的证书确认。设定一个强大并且复杂的密码（例如，密码长度为12个字符，其中包含大小写字母、数字和印刷字符）。尝试用这个密码的各种变化形式登陆，如删除最后一个字符，改变字符的大小写，或者删除任何特殊字符。如果其中一些尝试取得成功，继续系统性地尝试，了解完整的证书确认过程。

(4) 了解最小密码强度规则以及密码确认的程度后，在设法确定密码猜测攻击所需要使用的密码值范围，以提供攻击成功地可能性。尝试找出所有的内置账户，它们可能并不满足标准密码复杂度要求。

## 3. 测试用户名枚举
(1) 确定各种验证功能通过在屏幕上显示的输入字段、隐藏表单字段或Cookie提交用户名的每一个位置。这些位置通常存在于登陆、自我注册、密码修改、退出与账户恢复功能中。

(2) 向每个位置提交两个请求，其中分别包含一个有效和一个无效的用户名。分析服务器对每一个请求的响应的各方面细节，包括HTTP状态码、任何重定向、屏幕上显示的信息、任何隐藏在HTML页面源代码中的差异以及服务器做出响应的时间。请注意，其中一些差异可能极其细微（例如，看似相同的错误消息可能包含排版方面的细小差异）。可以使用拦截代理服务器的“历史记录”功能分析进出服务器的所有流量。WebScarab的一项功能可对两个响应进行比较，以迅速确定它们之间的任何差异。

(3) 如果从提交有效和无效用户名返回的响应中发现任何差异，那么使用另外一组用户名重复测试，确定响应之间是否存在相同的模式差异，以此作为自动化用户名枚举的基础。

(4) 检查应用程序中任何其他可帮助获得一组有效用户名的信息泄露源，例如，日志功能、注册用户列表以及在源代码注释中直接提及姓名或电子邮件地址的情况。

(5) 定位任何接受用户名的附属验证机制，并确定是否可以将其用于用户名枚举。特别注意允许指定用户名的注册页面。

## 4. 测试密码猜测的适应性
(1) 确定应用程序提交用户证书的每一个位置。通常，用户主要在主登陆功能和密码修改功能中提交证书。如果用户可提交任意用户名，密码修改功能才会成为密码猜测攻击的有效目标。

(2) 在每一个位置，使用一个受控制的账户手动提出几个包含有效用户名但证书无效的请求。监控应用程序的响应，确定他们之间的所有差异。如果应用程序经过大约10次登陆失败后还没有返回任何有关账户锁定的消息，再提交一个包含有效证书的骑牛。如果这个请求登陆成功，应用程序可能并未采用任何账户锁定策略。

(3) 如果没有控制任何账户，那么尝试枚举或猜测一个有效地用户名，并使用它提交几个无效的请求，监控任何有关账户锁定的错误消息。当然，应该意识到，这种测试可能会导致其他用户的账户被冻结或禁止。

## 5. 测试账户恢复功能
(1) 确定如果用户忘记他们的证书，应用程序是否允许他们重新控制自己的账户。通常，在主登陆功能附近有一个“忘记密码”链接即表示应用程序采用了密码恢复功能。

(2) 使用一个受控制的账户完成整个密码恢复过程，了解账户恢复功能的运作机制。

(3) 如果该功能使用机密问题之类的质询，确定用户是否可以在注册时设定或选择他们自己的质询。如果可以，使用一组枚举出的或常见的用户名获取一组质询，并对其进行分析，找出任何很容易猜测出答案的质询。

(4) 如果该功能使用密码“暗示”，采取和上个步骤相同的操作获取一组密码暗示，确定任何可轻易猜测出答案的暗示。

(5) 对账户恢复质询进行与主登陆功能相同的测试，确定可对其实施自动猜测攻击的漏洞。

(6) 如果该功能要求向用户发送一封电子邮件才能完成整个恢复过程，寻找任何可帮助控制其他用户账号的弱点。确定是否有可能控制接受以上电子邮件的地址。如果邮件内容中包含一个唯一的恢复URL，使用受控制的一个电子邮件地址获得若干邮件，尝试确定任何可帮助预测发布给其他用户的URL模式。

## 6. 测试“记住我”功能
(1) 如果主登陆功能或它的支持逻辑包含“记住我”功能，激活这项功能并分析它的作用。如果该功能允许用户随后不输入任何证书即可登陆，那么应该仔细分析这项功能，查找其中存在的所有漏洞。

(2) 仔细检查激活“记住我”功能时设定的所有持久性Cookie。寻找任何明确标识出用户身份或明显包含可预测的用户标识符的数据。

(3) 即使其中保存的数据经过严密编码或模糊处理，也要仔细分析这些数据，并比较“记住”几个非常类似的用户名和密码的结果，找到任何可对原始数据进行逆向功能的机会。

(4) 根据以上结果，适当修改Cookie的内容，尝试伪装成其他应用程序用户。

## 7. 测试伪装功能
(1) 如果应用程序包含任何明确的功能，允许一名用户伪装成另一名用户，那么仔细审查这项功能，查找所有允许未经正确授权即可伪装成任意用户的漏洞。

(2) 寻找所有用户提交的、用于确定伪装目标的数据。尝试修改这个数据，伪装成其他用户，特别是可帮助提升权限的管理用户。

(3) 当针对其他用户账户实施自动密码猜测攻击时，寻找所有明显使用多个有效密码的账户，或者几个使用相同密码的账户。这表示应用程序提供后门密码，以便管理员使用它时以任何用户的身份访问应用程序。

## 8. 测试用户名的唯一性
(1) 如果应用程序提供自我注册功能，允许指定想要的用户名，那么尝试使用不同的密码注册同一个用户名。

(2) 如果应用程序阻止第二个注册尝试，就可以利用这种行为枚举出注册用户名。

(3) 如果应用程序注册以上两个账户，深入分析这种情况，确定用户名与密码发生冲突时应用程序的行为。尝试修改一个账户的密码，使其与另一个密码相同。同时，尝试使用完全相同的用户名与密码注册这两个账户。

(4) 如果在用户名与密码发生冲突时，应用程序发出警报或产生一个错误，就可以利用这种行为实施自动化猜测攻击，确定其他用户的密码。针对一个枚举出的或猜测到的用户名，尝试使用这个用户名与不同的密码创建账号。应用程序拒绝某个特殊的密码即表示它可能是目标账户的现有密码。

(5) 如果应用程序接受相互冲突的用户名与密码，并且不产生错误，那么使用相互冲突的证书登陆，确定应用程序的行为，以及是否可以利用这种行为不经授权即可访问其他用户的账户。

## 9. 测试证书的可预测性
(1) 如果用户名或密码由应用程序自动生成，设法获得几个紧密相连的用户名或密码，确定任何可探测的顺序或模式。

(2) 如果用户名以可预测的方式生成，那么向后推导，获得一组可能有效地用户名。这些用户名可作为自动密码猜测与其他攻击的基础。

(3) 如果密码以可预测的方式生成，那么推导这种模式，获取应用程序向其他用户发布的一组密码。渗透测试员可以将这些密码与获得的用户名进行组合，实施密码猜测攻击。

## 10. 检测不安全的证书传输
(1) 遍历所有需要传输证书、与验证有关的功能，包括主登陆功能、账户注册功能、密码修改功能以及允许查看或更新用户个人信息的页面。使用拦截代理服务器监控客户端与服务器之间的所有流量。

(2) 确定在来回方向传输证书的每一种情况。可以在拦截代理服务器中设置拦截规则，标记包含特殊字符串的消息。

(3) 如果证书在URL查询字符串中传输，那么这些证书可能会通过浏览器历史记录、屏幕、服务器日志以及Referer消息头（如果访问第三方链接）泄露。

(4) 如果证书被保存在Cookie中，可能会通过XSS攻击或本地隐私攻击泄露。

(5) 如果证书被从服务器传送回客户端，攻击者就可以利用会话管理或访问控制漏洞，或者通过XSS攻击获取这些证书。

(6) 如果证书通过未加密连接传送，它们很可能被窃听者拦截。

(7) 如果使用HTTPS提交证书，但使用HTTP加载登陆表单，那么应用程序就容易遭受中间人攻击，攻击者也可能使用这种攻击手段获取证书。

## 11. 检测不安全的证书分配
(1) 如果应用程序通过某种带外通道创建账户，或者它提供的自我注册功能本身并不决定用户使用的全部初始证书，那么应该确定应用程序采用什么方法向新用户分配证书。常用的方法包括发送电子邮件，或者向邮政地址寄送信件。

(2) 如果应用程序生成以带外方式分配的账户激活URL，尝试注册几个紧密相连的新账户，并确定收到的URL中的顺序。如果能确定某种模式，努力预测应用程序发送给最近与后续用户的URL，并尝试使用这些URL占有他们的账户。

(3) 尝试多次重复使用同一个激活URL，看看应用程序是否允许这样做。如果遭到拒绝，尝试在重复使用URL之间锁定目标账户，看看URL是否仍然可用。确定使用这种方法是否可以给一个已经激活的账户设定一个新密码。

## 12. 检测不安全的存储
(1) 如果可以访问散列密码，应检查共享同一散列密码值的账户。尝试以采用最常用的散列值的密码登陆。

(2) 使用相关散列算法的离线彩虹表查找明文值。

## 13. 测试逻辑缺陷
### 13.1 故障开放条件
(1) 对于要求应用程序检查用户证书的每一项功能（包括登陆与密码修改功能），使用受控制的账户以正常方式访问这些功能。注意它们提交给应用程序的每一个请求参数。

(2) 连续多次重复以上过程，以各种无法预料的方式轮流修改每一个参数，破坏应用程序的逻辑。对每一个参数进行一下修改。
* 提交一个空字符串值。
* 完全删除名/值对。
* 提交非常长和非常短的值。
* 提交字符串代替数字或提交数字代替字符串。
* 以相同和不同的值，多次提交同一个命名参数。

(3) 仔细检查应用程序对上述请求的响应。如果出现任何无法预料的差异，对这个结果进行进一步测试。如果某个修改造成行为改变，设法将这个修改与其他更改组合在一起，推动应用程序的逻辑达到其限制。

### 13.2 测试多阶段处理机制
(1) 如果任何与验证有关的功能需要在一系列不同的请求中提交证书，确定每个阶段的主要目的，同时注意每个阶段提交的参数。

(2) 连续多次重复以上过程，修改提交请求的顺序，破坏应用程序的逻辑。相关测试包括：
* 以不同的顺序完成所有阶段，到达想要的那个阶段；
* 轮流直接进入每一个阶段，然后按正常的顺序访问后续步骤；
* 几次访问上述功能，轮流省略每一个阶段，然后在后一个阶段继续按正常的顺序访问；
* 根据观察到的结果及每个功能阶段的主要目的，尝试通过其他方式修改访问这些阶段的顺序，并访问开发者没有预料到的阶段。

(3) 确定是否有任何一项息息（如用户名）在几个阶段被提交，或者是因为用户提交了它几次，或者是因为它通过客户端在隐藏表单字段、Cookie或预先设置的查询字符串参数中传送。如果是这样，尝试在不同的阶段提交不同的值（包括有效和无效的值），并观察其后果。设法确定提交的数据是否是多余的，或者在一个阶段确认，随后即被应用程序信任，或者在不同的阶段通过不同的检查进行确认。尝试利用应用程序的行为获得未授权访问，或者降低多阶段机制实施的控制的效率。

(4) 寻找所有通过客户端传送的数据。如果应用程序使用隐藏参数在各个功能阶段中追踪进程的状态，那么攻击者就可以修改这些参数，从而破坏应用程序的逻辑。

(5) 如果进程的任何部分要求应用程序采用一个随机变化的质询，对它进行测试，查找一下两种常见的缺陷。
* 如果一个指定质询的参数与用户的响应一起提交，确定是否可以修改这个值，选择自己的质询。
* 多次使用相同的用户名处理上述不断变化的质询，确定每次是否出现一个不同的质询。如果每次的质询不相同，那么就可以重复进入这个阶段，直到应用程序显示希望的质询，以这种方式选择想要的质询。

## 14. 利用漏洞获取未授权访问
(1) 分析在各种验证功能中发现的所有漏洞，确定所有可在攻击应用程序过程中用于实现自己的目标的漏洞。通常，这包括尝试以另一名用户的身份进行验证；如果可能，以拥有管理权限的用户身份验证。

(2) 在实施自动攻击之前，留意已经确定的所有账户锁定防御。例如，当对登陆功能实施用户名枚举攻击时，在请求中提交一个常用的而不能完全随机的密码，以免在每一个发现的用户名上浪费一次登陆失败尝试。同样，应以广度优先而非深度优先的方式实施密码猜测攻击。首先使用单词列表中最常用的脆弱密码，然后使用其他值，对每一个枚举出的用户名实施密码猜测攻击。

(3) 构建在密码猜测攻击中使用的单词列表时，应考虑密码强度规则以及密码确认机制的完整性，避免使用不可能的或多余的测试密码值。

(4) 实施尽可能多的自动攻击，提高攻击的速度与效率。















