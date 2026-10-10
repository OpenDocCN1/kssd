

# .NET IL 语言直接操作线性地址。

> 原文：[`www.pediy.com/kssd/pediy10/101408.html`](https://www.pediy.com/kssd/pediy10/101408.html)

偶然间发现的， 原本是想用 IL 直接到线性地址去取数值，没想到真的可以。![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")IL 指令是我 HEX 手动改的。  挺有意思的，放出来和兄弟们分享下。  
  
以下这段指令输出 PE 文件的头的 4 个字节的值。   
  

代码:

```
.method /\*06000001\*/ private hidebysig static 
        int32  SquarePtrParam() cil managed
// SIG: 00 00 08
{
  // 方法在 RVA 0x2050 处开始
  // 代码大小       18 (0x12)
  .maxstack  2
  .locals /\*11000001\*/ init (\[0\] int32 t,
           \[1\] int32\* p,
           \[2\] int32 CS$1$0000)
  IL\_0000:  /\* 00   |                  \*/ nop
  IL\_0001:  /\* 20   | 00004000         \*/ ldc.i4     0x400000
  IL\_0006:  /\* E0   |                  \*/ conv.u
  IL\_0007:  /\* 0B   |                  \*/ stloc.1
  IL\_0008:  /\* 07   |                  \*/ ldloc.1
  IL\_0009:  /\* 4A   |                  \*/ ldind.i4
  IL\_000a:  /\* 0C   |                  \*/ stloc.2
  IL\_000b:  /\* 08   |                  \*/ ldloc.2
  IL\_000c:  /\* 00   |                  \*/ nop
  IL\_000d:  /\* 00   |                  \*/ nop
  IL\_000e:  /\* 00   |                  \*/ nop
  IL\_000f:  /\* 00   |                  \*/ nop
  IL\_0010:  /\* 00   |                  \*/ nop
  IL\_0011:  /\* 2A   |                  \*/ ret
}

```