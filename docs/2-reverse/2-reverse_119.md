

# OllyHTML 脚本详解（一）结构

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-357.htm`](https://www.pediy.com/kssd/pediy08/pediy8-357.htm)

【文章标题】: OllyHTML 脚本详解（一）结构  
【文章作者】: dreaman  
【作者邮箱】: dreaman\_163@163.com  
【作者主页】: http://dreaman.haolinju.net  
【软件名称】: OllyHTML  
【软件大小】: 800KB  
【下载地址】: http://dreaman.haolinju.net  
【加壳方式】: 无  
【保护方式】: 无  
【编写语言】: c++  
【使用工具】: vc7.1  
【操作平台】: win2000 以上系统  
【软件介绍】: Ollydbg 的插件，用以支持用 DHTML 作脚本  
【作者声明】: 个人觉得用 DHTML 作脚本是能做一些比较复杂的事情的，希望大家能用用：）  
  
写完 OllyHTML 与 IdaHTML 插件后一直希望能看到有人能用上它，没想到事与愿违，因为我自己的破解功底很浅，所以又写不出好的脚本来，  
但我又感觉用 HTML+javascript 的方式是可以适用于需求变化频繁然而又确实有规律可循的场合的（我想破解与逆向算是这种场合的一种吧）  
，这便有了详细介始一下 OllyHTML 脚本的想法，希望我能够把它说明白，更大的愿望是希望能有人用它写点什么，比如把自己的破解与逆向  
的经验表达成一些脚本：）  
  
本教程计划分为五个主要部分：  
  OllyHTML 脚本详解（一）结构  
  OllyHTML 脚本详解（二）基本原理  
  OllyHTML 脚本详解（三）静态分析 API  
  OllyHTML 脚本详解（四）动态调试 API  
  OllyHTML 脚本详解（五）其它 API  
主要以介始 OllyHTML 插件提供的 API 为主，配以少量的例子说明 API 的用法，因为 OllyHTML 插件的 API 主要是对 Ollydbg 插件 API 的封装，所以有许  
多 API 能直接对应到 Ollydbg 的插件 API 上，这样的 API 的说明都是按我的理解所写，错误之处还望大家海涵：）我在 IdaHTML 插件中也加了调试功  
能，不过感觉 Ida Pro 对调试的支持还是没法与 OD 相提并论，而且也不稳定，不过 OD 的静态分析没有 Ida Pro 强大，因为 OllyHTML 与 IdaHTML 插件  
是基于同样的内核实现的，它们之间是可以通信的，如果写完上面说的五个部分还有空并且能找到需要结合 OD 与 IdaPro 的实例，偶可能会加一个  
OllyHTML 与 IdaHTML 协作的部分，这是后话，暂且搁下吧。  
今天是第一部分，偶是第一次写教程，所以请大家不要拍砖：）  
  
按照一般教程的做法，我们首先来一个 Hello world，因为是要写脚本，所以我们假设您已经了解 HTML 与 javascript 语言的简单用法。  
  
    <html>  
      <head>  
        <title>Hello world</title>  
        <script src="res://OllyHTML.dll/const.js"></script>  
      </head>  
      <body>  
        <table style="width:100%;height:100%">  
          <tr>  
            <td>  
              <textarea id="resultArea" style="width:100%;height:100%" rows="10"></textarea>  
            </td>  
          </tr>  
          <tr style="height:24px">  
            <td>  
              <input type="button" value="执行" onclick="doStart()">  
            </td>  
          </tr>  
        </table>  
        <script>  
          function doStart()  
          {  
            resultArea.value="Hello , World !";  
          }  
        </script>  
      </body>  
    </html>  
        
  
将上面这段脚本保存成 test.htm，然后用 OllyHTML=>Load=>Html，指定文件 test.htm，就会看到脚本装入后的样子了，点一下上面的“执行”  
按钮，就会看到上面的文本显示区出现：Hello , World !  
  
(对大多数人看到这里就可打住了，请直接看下一篇！)    
  
这个脚本其实与 OllyHTML 没有太大关系（唯一有关的是引用了插件的常量资源文件），只是一个普通的 HTML 网页，不过它展示了 OllyHTML  
脚本的主要框架。  
  
脚本首先是一个 HTML 文件，所以它的最外层架子是这样的：  
  
    <html>  
      <head>  
      </head>  
      <body>  
      </body>  
    </html>  
  
也就是有一个 HTML 头部与一个 HTML BODY。  
  
HTML 头部主要用来放置标题，样式表，脚本引用之类，我们的脚本里是一个标题与一个插件常量文件引用，标题的写法如下：  
    <title>Hello world</title>  
您只需要将其中的 Hello world 替换为您自己的脚本标题就可以了。插件常量文件引用的写法总是这样的，在您的脚本里原样拷贝就可以了：  
    <script src="res://OllyHTML.dll/const.js"></script>  
  
HTML BODY 部分可以放置用户可见的各种元素与用户脚本（脚本也可以放在 HTML 头里，不过放在 BODY 的尾部可以更好的访问可见元素），  
用户可见的元素总是需要布局的，所以用户可见的部分嵌在一个 TABLE 里面，我们想像一下一个脚本通常需要与用户交互的东东：  
1、执行信息显示  
2、执行用户命令的按钮  
我们的 hello world 里就是只用了这两个简单的东东：  
  
    <table style="width:100%;height:100%">  
      <tr>  
        <td>  
          <textarea id="resultArea" style="width:100%;height:100%" rows="10"></textarea>  
        </td>  
      </tr>  
      <tr style="height:24px">  
        <td>  
          <input type="button" value="执行" onclick="doStart()">  
        </td>  
      </tr>  
    </table>  
      
我们用了一个二行一列的 TABLE 来布局我们的两个与用户交互的元素，用来显示信息的我们用的是 TEXTAREA，它是一个多行文本控件，  
我们给了它一个 ID：resultArea，这就相当于在脚本里定义了一个名为 resultArea 的变量，它的类型就是 TEXTAREA,不了解这些也没关系，  
因为我们差不多总是只用到一个访问它的方法：  
  
    resultArea.value  
      
执行用户命令的按钮如下：      
  
    <input type="button" value="执行" onclick="doStart()">  
      
我们给它一个显示名“执行”，然后为它指定一个当用户点击时要调用的脚本函数:  
      
    doStart()  
      
上面的 UI 虽然简单，但我们的大多数脚本只需要它们就足够了，当然，您对 DHTML 很熟的话，您可以做得很炫：）  
  
接下来是脚本的主体了，脚本总是嵌在一个这样的标签里：  
  
    <script>  
    </script>  
    
好了，现在关于 HTML 的部分就说完了，下面是关于 javascript 语言的了。  
javascript 语言是相当灵活与强大的，不过我们现在只说最简单的用法，也就是如何写一个前面 UI 里指定的当用户点击时要调用的函数，  
它是这样的：  
    
    function doStart()  
    {  
      resultArea.value="Hello , World !";  
    }  
  
上面我们定义了当用户点击“执行”按钮时调用的函数，函数名为 doStart，它没有参数，函数体只有一条语句，它将“hello, world !”字符  
串赋给 reaultArea.value，也就是显示信息给用户的那个控件。  
javascript 是一种类 C 语言，它的主要语句与操作都与 C 相同，主要的不同是 javascript 不是强类型的，也就是 javascript 里的所有变量都  
不需要指定类型，我们定义一个变量时只需要这样：  
  
    var varName;  
      
或者，在定义变量的同时给变量指定一个初始值：  
      
    var varName=0;  
  
其它的我们就不多说了，因为我们主要目的是写脚本，这一篇主要是为了说明一个脚本像什么样子，呵呵，有关 HTML 与 javascript 的内容，  
可以参考 MS 的 MSDN 的 WEB 编程部分或者上网随便搜一下，遍地都是。  
  
(估计有人开始大呼上当了，这搞什么嘛，明白的不看就明白，不明白的看 N 遍还是不明白！下回我们开始入正题，着重介始 OllyHTML 为脚本  
提供的对象模型。)  
  
附带的提一下 javascript 语言的一类简写语法，这些语法被广泛用于 ajax 技术中，这就是 JSON （JavaScript Object Notation），我们  
看一下其中的对象与数组的简写语法。  
  
1、对象是名字/值对的集合，名字与值用冒号分隔，每个名字/值对间用逗号分隔，然后整个对象用花括号包含。比如：  
var obj=  
{  
  data:"test",  
  getData:function()  
  {  
    return this.data;  
  }  
}  
这便定义了一个对象实例变量 obj，这个对象有一个数据成员 data，它的值是"test"，它还有一个函数成员 getData,执行它返回数据成员的值。  
  
2、数组是值的集合，值之间用逗号分隔，然后整个数组用方括号包含。比如：  
var datas=\[1,2,3,4\];  
这便定义了一个数组变量 datas，它是由 1，2，3，4 四个整数构成的数组。  
  
3、对象定义与数组定义中的值可以是 javascript 语言里的字符串、数字、对象（函数也是一种对象，所以也包括函数）、数组等，这样对象与数  
组的定义可以是自嵌套的，比如可以这样写一个数组：  
var ds=\[\[11,12\],\[21,22\]\];  
这是一个有两个元素的数组，每个元素又是一个有两个元素的数组。也可以定义一个嵌套的对象：  
var obj=  
{  
  obj1:  
  {  
    t11:1,  
    t12:2  
  }  
  obj2:  
  {  
    t21:1,  
    t22:2  
  }  
}  
这样定义的对象实例 obj 有两个数据成员 obj1、obj2，这两个数据成员分别是两个对象实例。  
  
最后提一下 javascript 的调试，VS.NET 是个不错的调试器，不过因为我们的脚本是嵌入在 ollydbg 里面执行的，没办法直接用 vs.net 装入，所以只  
能用 attach 方式来调试，ms 为它的 javascript 实现加了一个语句:debugger，相当于 win32 程序里的 API DebugBreak，我们在脚本里要设断的地方写  
上  
debugger;  
然后运行，就会中断并提示选择调试器，然后在 vs.net 里就可以单步跟踪每条语句、查看各个变量的值、执行堆栈等等了。  
  
  
\--------------------------------------------------------------------------------  
  
                                                       2006 年 09 月 05 日 10:01:39 

 4dd466d0e1e0cc123afbe504da2462ca