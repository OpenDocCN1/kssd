
# eXeScope 6.50 本地溢出分析

> 原文：[`www.pediy.com/kssd/pediy10/84975.html`](https://www.pediy.com/kssd/pediy10/84975.html)

eXeScope 6.50 本地溢出分析

无聊逛了下 `http://www.milw0rm.com` 看最近有什么 ODAY 没，顺便挑一个分析下玩玩。

于是就看到了这个

`http://www.milw0rm.com/exploits/8270`

是 `eXeScope` 6.50 `Local` `Buffer` `Overflow` `Exploit`。

用 `eXeScope` 6.50 打开生成的 `example.exe` 后弹出了计算器。

看了下他的 exploit，感觉比较好理解。为了能找到漏洞发生的原因，我把 shellcode 前面的 `90` 改为 `cc` `(` 我是在 `example.exe` 文件偏移 `11e8` 处修改的 `)` ，也就是 `int3`。然后用 `od` 载入 `eXeScope` ,再用 `eXeScope` 打开 `example.exe`，最下面显示堆栈溢出，使用 `shift+f7`/`f8`/`f9` 来忽略程序异常。`shift+f9` 执行后就来到这里。

可以看到程序暂停在了 `004d3258` 处。下面接着就是 shellcode 了。可是这个溢出的根本原因是什么呢？

看堆栈处，

`004069FA` 返回到 `eXeScope.@LoadResString+36` 来自 `<eXeScope.LoadStringA>`

看来程序是经过了这个函数后来到此处的。

重新加载一遍，`ctrl+g` 来到 `004069fa` 处，在 `LoadStringA` 函数上面也就是 `004069f4` 处下个断点，然后 `f7` 走进去，这时发现

```
FF25 54324D00 JMP DWORD PTR DS:[<&user32.LoadStringA>] ; LoadStringA
DS:[004D3254]=004D3258 (<&user32.MessageBoxA>)
```

本地调用来自 `@LoadResString+31`

导入表被修改了，因为只要执行到 `004D3258` 就 OVER 了。

看来这个漏洞是覆盖了 `eXeScope` 的导入表。

不过还没有找到在哪里覆盖了。

还是用 `windbg` 再看看吧。先打开 `eXeScope`，接着在命令行下

```
C:\\Program Files\\windbg>cdb -pn eXeScope.exe
```

输入 `G` 运行，再用 `eXeScope` 打开 `example.exe`，

`eXeScope+0x23b6`:

```
004023b6 8b1e mov ebx,dword ptr [esi] ds:0023:4141413d=????????
```

这里有个内存读取错误。

再输入 `G` 运行

```
(778.6a8): Break instruction exception - code 80000003 (first chance)
eax=00400000 ebx=00407dfc ecx=00000000 edx=004cd0c8 esi=0012f778 edi=00000000
eip=004d3258 esp=0012f1d8 ebp=0012f77c iopl=0 nv up ei pl nz ac po nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000212
eXeScope+0xd3258:
004d3258 cc int 3
```

这里 `eip` 指向 `004d3258` 了。

看来关键点就是 `004023b6`。这里内存读取错误后，应该有个异常处理的，然后程序才能正常执行下去。

先看 `004023b6` 吧。

先运行起来，在打开 `example.exe` 的一瞬在 `004023b6` 下断点，断下后

```
.dd 4d3000
004D3000 00000000
004D3004 00000000
004D3008 00000000
004D300C 000D393C
004D3010 000D31A4
004D3014 00000000
004D3018 00000000
004D301C 00000000
004D3020 000D3C2A
004D3024 000D3250
```

发现导入表现在还没有被覆盖，而我们要找的就是看导入表在哪一刻被覆盖的。

那我就在 `4d3000` 处下内存写入断点。

然后把 `004023b6` 处断点取消。`F9` 运行之就来到下面了

```
/*4C70A4*/ PUSH 0
/*4C70A6*/ MOVZX EAX,WORD PTR DS:[4D20EE]
/*4C70AD*/ MOV ECX,EAX
/*4C70AF*/ SHL ECX,3
/*4C70B2*/ LEA ECX,DWORD PTR DS:[ECX+ECX*4]
/*4C70B5*/ MOV EDX,eXeScope.004D2220
/*4C70BA*/ MOV EAX,eXeScope.004D1D8C
/*4C70BF*/ CALL eXeScope.0040302C
/*4C70C4*/ CALL <eXeScope.@@_IOTest>
/*4C70C9*/ MOV EAX,eXeScope.004C75E8
/*4C70CE*/ CALL <eXeScope.@TextToShortCut_0>
/*4C70D3*/ MOV DWORD PTR DS:[4D2C48],EAX
/*4C70D8*/ MOV EAX,DWORD PTR DS:[4D2C78]
/*4C70DD*/ CALL eXeScope.004027BC
```

从 OD 注释上看应该是处理资源的时候出的问题。

好，重新来过，在稍微前面点 `4C70A4` 处下个断点。经测试发现只要过了

```
/*4C70BF*/ CALL eXeScope.0040302C
```

导入表就被覆盖。看来就是 `call 0040302C` 出的问题

`F7` 进去仔细看个明白吧。

进去后，又经我人肉测试，发现过了

```
/*402FBA*/ PUSH 0
/*402FBC*/ LEA EAX,DWORD PTR SS:[EBP-4]
/*402FBF*/ PUSH EAX
/*402FC0*/ MOV EAX,DWORD PTR DS:[EBX+8]
/*402FC3*/ IMUL ESI
/*402FC5*/ PUSH EAX
/*402FC6*/ PUSH EDI
/*402FC7*/ MOV EAX,DWORD PTR DS:[EBX]
/*402FC9*/ PUSH EAX
/*402FCA*/ CALL DWORD PTR SS:[EBP+C]
/*402FCD*/ TEST EAX,EAX
```

过了 `/*402FCA*/ CALL DWORD PTR SS:[EBP+C]` 后导入表就被覆盖

OK，现在问题终于找到了。

就是 `/*402FCA*/ CALL DWORD PTR SS:[EBP+C]` 的原因。

可以看到这个函数是 `<JMP.&kernel32.ReadFile>`

他的几个参数分别是

句柄：`140`

读文件的缓冲区地址 `004d2220`

读的字节大小：`00002d78`

：

而导入表起始地址是 `004d31a4`.

`004d2220+00002d78=4d4f98`

显然超过了导入表的地址，全部覆盖了。

然后作者利用最近使用的导入表函数的地方，也就是 `LoadStringA`，替换成了 `004d3258`,刚好指向 shellcode 地址。

综上所述，该溢出的根本原因就是 `/*402FCA*/ CALL DWORD PTR SS:[EBP+C]`

处 `<JMP.&kernel32.ReadFile>` 在读取资源时没有限制大小，导致构造的数据覆盖了 `eXeScope` 的导入表。

最后我不得不佩服作者如此巧妙的利用。另外这个漏洞真有趣,因此记录之.OVER。

附件是 POC 利用程序,有兴趣的可以测试下自己的 eXeScope.

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `example.rar` |
