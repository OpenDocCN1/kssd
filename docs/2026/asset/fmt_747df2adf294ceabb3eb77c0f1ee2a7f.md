
```
001B:0CE34327  681541E30C          PUSH      0CE34115           //此处有段清零代码
001B:0CE3432C  C3                  RET                          //先去清零，再去fake OEP
```

通过分析上面的花指令单步跟踪，重建后得到的代码如下，运行正常。

```
push    ebp
mov     ebp, esp
push    FF
push    54E948
push    542B94
lea     eax, [ebp+20]
mov     [ebp-10], eax
mov     dword ptr [ebp-6C], 7FFDF000
xor     eax, eax
mov     [ebp-70], eax
mov     [ebp-74], eax
leax    eax, [ebp-74]
mov     [ebp-18], eax
jmp      52F07B
```

由于可以“机械化”操作，实际上是可以写个工具自动或者半自动重建`stolen bytes`的。

BTW: `aspr 1.23 RC4`把虚拟地址弄得很大，`full dump`出来的文件有100多M，幸好`rebuild`一下就变小了，不然要每个`section`单独dump然后再合成。
