
# VProtect1.XX-1.84 unpacker

> 原文：[`www.pediy.com/kssd/pediy12/130135.html`](https://www.pediy.com/kssd/pediy12/130135.html)

【文章标题】: `VProtect1.XX-1.84 unpacker`

【文章作者】: `wuqing1501(笨笨鼠)`

【作者邮箱】: 木有

【作者主页】: 木有

【作者 QQ 号】: 木有

【下载地址】: 自己搜索下载

【保护方式】: `VProtect` 默认加壳

【使用工具】: 老三样

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 详细过程

在这里呢首先我先要感谢感谢 `NOOBY`、`KISSY` 和 `COOOLIE`，因为 `KISSY` 和 `nooby` 我学会了脱 `VMP` 用 `DLL` 修复的方法，通过研究 `NOOBY.DLL` 和 `kissy.dll` 我明白了这个 `DLL` 的原理，所以才有这篇文章。

今天说下 `VProtect1.84` 以下版本的脱壳方法，其实我感觉脱 `VP` 的话，`nooby` 在脚本区公布的那个主程序 `1.82` 的脱壳脚本可以说是相当的无敌，应该可以通杀 `1.82` 以下所有的版本，但是对于高版本的话，可能需要一些修改，但是这个我没有研究过所以在这里就不多说了。我现在说另外一种方法，相信大家都看过 `kissy` 牛脱 `VMP` 的视频吧?其实我的方法就是学习 `kissy` 牛还有 `NOOBY` 牛脱 `VMP` 用 `DLL` 修复 `IAT` 的方法，但是 `VP` 和 `VMP` 有区别，因为 `VMP` 在 `IAT` 的处理过程中，只使用了一种算法就是加法，而 `VP` 为了防止 `kissy.dll` 还有 `nooby.dll` 的使用，所以在 `1.84` 以及以前的版本中都使用了三种算法，那就是：相加、异或、取反。所以我们直接用脱 `VMP` 的方法去脱 `VP` 是不行的，那两个 `DLL` 的算法都是加法，所以我们要自己写个 `DLL`，其实这个 `DLL` 的原理，相信使用过 `DLL` 修复的人都明白的原理还是很简单的，但是我是个菜鸟，不会编程，用易语言写那个 `DLL` 最后总是因为有个问题没有解决最后没有成功，最后还是拿到了 `kissy.dll` 的源码进行了修改，在这里我先十分感谢 `KISSY` 和 `COOOLIE`,所以我们现在要做的工作就是：修改 `KISSY` 脱 `VMP` 的脚本，修改 `COOOLIE` 的 `kissy.dll` 的源码。

在这里我不公开 `kissy.dll` 的源码，只公开成品还有脚本，因为那个源码我基本上只修改了一下那个算法的地方所以其他的东西都还是原来的，在这里还请 `KISSY` 牛海涵。

### 脚本

代码:

```
  /*
  /////////////////////////////////////////////////////////////////////////
  脚本   : VP1.XX-Vprotect 1.84 Unpacker
  调试选项 : 1.设置 OllyDbg 忽略所有异常选项  
       2.使用海风月影全选项.
       3.去掉'选项-调试设置-地址-解析修饰符号名称' 前面的勾,然后重新打开 OD.跑脚本.(必须)
            4.使用 PhantOm 保护 DRX。
  工具 : OllyDbg1.1, ODBGScript 1.65
  感谢 : 感谢 KISSY 的脚本，感谢苦力的 DLL 源码，感谢那些曾经帮助我的人!
  ////////////////////////////////////////////////////////////////////////////////
  
  var getapi
  var write
  var addr
  var dword
  var vpro
  var rapi
  var dllname
  var apiname
  var imgbase
  var imgbasefromdisk
  var sizeofimg
  var tmp1
  var tmp2
  var tmp3
  var secbase
  var secsize
  var logfile
  var oep
  var fixtype
  var typeadd
  var typexor
  var typenot
  var vmiataddr
  mov logfile,"FakIat.txt"
  
  
  /////////////////////////
  //配置区
  /////////////////////////
  mov vmexit,008D21ED      //VM 出口
  mov oep,0069C168         
  mov write,008AC492       //填充函数
  mov typeadd,008A8F41     //对 IAT 进行 SUB 处理的地址
  mov typexor,008AA673     //对 IAT 进行 xor 处理的地址
  mov typenot,008AC257     //对 IAT 进行 not 处理的地址
  /////////////////////////
  
  cmp $VERSION, "1.64"
  jb odbgver
  
  GMI eip, MODULEBASE
  mov imgbase, $RESULT
  mov tmp1, \[imgbase+3C\]
  add tmp1, imgbase
  mov signVA, tmp1
  mov imgbasefromdisk, \[signVA+34\]
  mov sizeofimg, \[signVA+50\]
  mov tmp1, signVA
  add tmp1, f8
  mov tmp2, 0
  mov tmp2, \[signVA+6\], 2
  wrta logfile,tmp2
  wrta logfile,"\\r\\n"
  
  last:
  mov secsize, \[tmp1+8\]
  mov tmp3, \[tmp1+0C\]
  add tmp3, imgbase
  mov secbase, tmp3
  wrta logfile,secbase
  wrta logfile,"\\r\\n"
  wrta logfile,secsize
  wrta logfile,"\\r\\n"
  cmp tmp2, 1
  je lab1
  add tmp1, 28
  sub tmp2, 1
  jmp last
  
  lab1:
  
  
  cmp imgbasefromdisk, imgbase
  je lab1\_1
  jmp error
  
  lab1\_1:
  bc
  
  start:
  bp vmexit
  cmp eip,oep
  je exit
  cmp eip,write
  je findkey
  cmp eip,typeadd
  je findkey
  cmp eip,typexor
  je findkey
  cmp eip,typenot
  je findkey
  esto
  len \[ecx\]
  cmp $RESULT,8
  jb start
  scmpi \[ecx\],"ole32.dll"
  je loopfix
  scmpi \[ecx\],"advapi32.dll"
  je loopfix
  scmpi \[ecx\],"user32.dll"
  je loopfix
  scmpi \[ecx\],"kernel32.dll"
  je loopfix
  cmp \[ecx\],"oleaut32.dll"
  je loopfix
  scmpi \[ecx\],"advapi32.dll"
  je loopfix
  cmp \[ecx\],"uxtheme.dll"
  je loopfix
  scmpi \[ecx\],"ws2help.dll"
  je loopfix
  cmp \[ecx\],"ws2\_32.dll"
  je loopfix
  scmpi \[ecx\],"winspool.dll"
  je loopfix
  cmp \[ecx\],"MSCTFIME.dll"
  je loopfix
  scmpi \[ecx\],"shell32.dll"
  je loopfix
  cmp \[ecx\],"shlwapi.dll"
  je loopfix
  scmpi \[ecx\],"GDI32.dll"
  je loopfix
  cmp \[ecx\],"rpcrt4.dll"
  je loopfix
  scmpi \[ecx\],"secur32.dll"
  je loopfix
  cmp \[ecx\],"msvcrt.dll"
  je loopfix
  scmpi \[ecx\],"version.dll"
  je loopfix
  cmp \[ecx\],"comctl32.dll"
  je loopfix
  scmpi \[ecx\],"iphlpapi.dll"
  je loopfix
  cmp \[ecx\],"imagehlp.dll"
  je loopfix
  scmpi \[ecx\],"psapi.dll"
  je loopfix
  cmp \[ecx\],"msctf.dll"
  je loopfix
  scmpi \[ecx\],"msimg32.dll"
  je loopfix
  cmp \[ecx\],"imm32.dll"
  je loopfix
  scmpi \[ecx\],"comdlg32.dll"
  je loopfix
  scmpi \[ecx\],"ntdll.dll"
  jne start
  
  loopfix:
  esto
  bp typeadd
  bp typexor
  bp typenot
  bp oep
  len \[ecx\]
  cmp $RESULT,0 
  je loopfix
  sub $RESULT,4
  readstr \[ecx\],$RESULT
  mov dllname,$RESULT
  len \[edx\]
  cmp $RESULT,0 
  je loopfix
  readstr \[edx\],$RESULT
  mov apiname,$RESULT
  esto
  esto
  esto
  esto
  
  add eax,ecx
  add eax,7
  mov addr,eax
  mov dword,\[addr+7\]
  sub eax,7
  sub eax,ecx
  cmp dword,0
  je start
  mov fixtype,0
  wrta logfile,addr
  wrta logfile,","
  wrta logfile,dword
  wrta logfile,","
  wrta logfile,dllname
  wrta logfile,","
  wrta logfile,apiname
  wrta logfile,","
  wrta logfile,fixtype
  wrta logfile,"\\r\\n"
  jmp start
  
  
  goon:
  cmp eip,oep
  je exit
  bp typeadd
  bp typexor
  bp typenot
  bp write
  bp oep
  esto
  
  findkey:
  cmp eip,oep
  je exit
  cmp eip,typeadd
  je addkey
  cmp eip,typexor
  je xorkey
  cmp eip,typenot
  je notkey
  jmp goon
  
  addkey:
  mov fixtype,0
  mov dword,\[eax+ecx+10\]
  gn edx
  mov dllname,$RESULT\_1
  mov apiname,$RESULT\_2
  cmp $RESULT,0 
  je goon
  bc eip
  bp write
  esto
  bc eip
  mov addr,edx
  jmp writetxt2
  
  xorkey:
  mov fixtype,2
  mov dword,\[eax+edx+10\]
  gn ecx
  mov dllname,$RESULT\_1
  mov apiname,$RESULT\_2
  cmp $RESULT,0 
  je goon
  bc eip
  bp write
  esto
  bc eip
  mov addr,edx
  jmp writetxt2
  
  
  notkey:
  mov fixtype,1
  mov dword,edx
  gn edx
  mov dllname,$RESULT\_1
  mov apiname,$RESULT\_2
  cmp $RESULT,0 
  je goon
  bc eip
  bp write
  esto
  bc eip
  mov addr,edx
  
  writetxt2:
  wrta logfile,addr
  wrta logfile,","
  wrta logfile,dword
  wrta logfile,","
  wrta logfile,dllname
  wrta logfile,","
  wrta logfile,apiname
  wrta logfile,","
  wrta logfile,fixtype
  wrta logfile,"\\r\\n"
  jmp goon
  
  
  error:
  msg "dll 不支持."
  RET
  
  odbgver:
  msg "ODSCR 版本要大于 1.65"
  ret
  
  exit:
  bc
  bphwc
  ret
```

脚本我就不多说了，这里要说明一下配置区的几个参数。

#### 1、vmexit

也就是 `VP` 的 `VM` 出口，对于 `1.93` 以下的版本，`VP` 加壳出来的 `VM` 出口基本上都是固定的。`VM` 出口代码如下：代码:

```
  58            POP EAX                             
  8903          MOV DWORD PTR DS:\[EBX\],EAX
  8B07          MOV EAX,DWORD PTR DS:\[EDI\]
  8B5F 0C       MOV EBX,DWORD PTR DS:\[EDI+C\]
  8B4F 04       MOV ECX,DWORD PTR DS:\[EDI+4\]
  8B57 08       MOV EDX,DWORD PTR DS:\[EDI+8\]
  8B6F 14       MOV EBP,DWORD PTR DS:\[EDI+14\]
  8B77 18       MOV ESI,DWORD PTR DS:\[EDI+18\]
  FF77 24       PUSH DWORD PTR DS:\[EDI+24\]
  9D            POPFD
  8B67 10       MOV ESP,DWORD PTR DS:\[EDI+10\]
  C747 50 0000000>MOV DWORD PTR DS:\[EDI+50\],0
  8B7F 1C       MOV EDI,DWORD PTR DS:\[EDI+1C\]
  C3            RETN  
```

二进制：

`58 89 03 8B 07 8B 5F 0C 8B 4F 04 8B 57 08 8B 6F 14 8B 77 18 FF 77 24 9D 8B 67 10 C7 47 50 00 00 00 00 8B 7F 1C C3`

这段代码基本上是固定的所以要找 `VM` 出口的话 这段代码中的任意一些指令都可以作为 `VM` 出口的特征码。我们需要找的 `VM` 出口就是那个 `RETN` 的地方。

#### 2、oep

对于 `VP` 加壳的程序，寻找 `OEP` 的话我在以前的文章也有说过，`NOOBY` 牛的脱壳脚本中也有方法，我这里再说一个就是对 `VirtualProtect` 函数下断（注意 `VP` 的检测不要在段首下 `F2`，可以在段尾或者中间 `F2`），`F9` 运行，然后取消 `VirtualProtect` 断点，`CODE` 段下内存访问断点，`F9` 运行一般都可以到 `OEP`!

#### 3、write

其实就是相当于 `VMP` 中的填充函数，代码如下代码:

```
  8902            MOV DWORD PTR DS:\[EDX\],EAX
```

至于这段代码怎么找的话，可以在代码中搜索所有的这个二进制，然后判断一下哪句代码是我们需要的当然这样是很麻烦的，说下稍微简单一点的方法吧。我们可以先找其他的特征码也就是下面的对 `IAT` 处理的代码都可以作为特征码，而且这个特征码一般在 `VP` 加壳的程序中只有一句，找到后我们下段，然后然后运行程序停下来后就是处于 `IAT` 处理的代码中，然后我们可以学习 `NOOBY` 牛的方法 `CTRL+T` 在最后一项中打勾，并输入我们需要找的代码

`MOV DWORD PTR DS:\[EDX\],EAX`

`NOT EDX`

然后 `CTRL+F11` 就可以了这样停下来后我们就可以找到这个填充函数了。

#### 4、typeadd typexor typenot



### VP 的 IAT 处理算法

在这里我们要先知道 `VP` 是如何处理 `iat` 的。这个我在很早以前发过类似的文章都有说明的，这里就不在说了。我只说对于 `IAT` 的处理算法，脚本中原来只有一个算法，所以只要知道填充地址和填充值，我们就可以用 `DLL` 计算出来 `API`。但是 `VP` 有三种算法，所以我们要先搞清楚 `VP` 是如何处理 `IAT` 的，代码如下：

```assembly
typeadd :
    2B5401 10       SUB EDX,DWORD PTR DS:[ECX+EAX+10]
typexor :
    334C10 10       XOR ECX,DWORD PTR DS:[EAX+EDX+10]
typenot :
    F7D2            NOT EDX
```

也就是 `VP` 中进行 `IAT` 处理的算法的时候用的就是这三条语句，所以我们需要在加壳的程序中找到这三条语句就可以了。

- 对于 `ADD` 和 `XOR` 的两种很好找，因为一般情况下，`VP` 这个版本加壳的程序中这两句代码都只有一句，直接搜二进制就可以了。
- 但是对于 `NOT` 类型的不好找，不过我们可以使用找 `WRITE` 参数的方法一样找到。

### 脚本运行与 DLL 修复经验

找到了这些参数基本上就算是完工了，可以跑脚本了。在这里说下我自己的一点经验：

1. 如果用 `DLL` 修复 `VP` 也好 `VMP` 也好，使用脚本插件的时候一定要使用 `1.65` 或者比 `1.65` 稍微大一点点比如 `1.67` 的版本，不要使用高版本！高版本生成的都是多行的，处理起来会非常困难。
2. 如果使用 `1.65` 版本的话，生成的 `IAT` 文件中就会有很多的“黑块”（这个黑块就是空格）。有这个的存在，`DLL` 就不能正常运行，所以我们在这里要处理“黑块”。
3. 对于处于每行中最后的几个黑块是不影响 `DLL` 使用的。影响 `DLL` 的是每行中间和“，”在一块的那个黑块。
4. 我们可以使用文本编辑器中的替换功能，具体你使用什么软件都可以，只要它能识别出 `IAT` 文件中的黑块就可以。可以同时选中“黑块”和“，”然后在替换框那里拷贝进去，而要替换的字符那里就填写“，”。这样的话就可以把所有的影响使用的黑块全都清除掉了。

### 脱壳后处理

最后因为 `VP1.8X` 版本貌似为了 `ANTIdump` 会申请三个大小为 `8000` 区段用来计算 `IAT`，所以脱壳后要 `dump` 这三个区段补到脱壳后的程序中。修正 `OEP` 和 `CODE` 段修正为可写。

至此，基本上所有的功能就算完成了，给脱壳后的文件添加 `DLL` 就可以了！

在这里我就不提供样品了，大家可以拿 `DEMO` 主程序测试！

### 资源下载

脚本和 `DLL`：

*   [`u.115.com/file/f58c2d8b07#`](http://u.115.com/file/f58c2d8b07#)
*   `VProtect1.XX-1.84_unpacker.rar`

### 经验总结

1. 多动手分析比看文章强百倍。
2. 以前脱壳修复 `IAT` 的时候只要找 `RETN` 或者返回找到真实的 `API` 填充到需要修复的地方就可以了，而这次这个方式却要分析 `IAT` 的加密算法。`VP1.84` 以下版本的算法简单，没有那么多的干扰和垃圾代码；对于高版本的就不一样了，很多的垃圾代码干扰分析，所以整个分析下来很累，但是对于分析算法来说还是让自己学到不少的东西！
3. 谢谢 `KISSY`、`NOOBY`、`COOOLIE` 还有曾经帮助过我的所有人！

### 版权声明

本文原创,小菜一个, 转载请注明作者并保持文章的完整, 谢谢!

2011 年 03 月 01 日 10:40:16

### 上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | | `VProtect1.XX-1.84 unpacker.rar` |
|---|---|---|
