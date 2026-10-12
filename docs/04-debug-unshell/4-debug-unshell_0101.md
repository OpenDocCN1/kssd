
# 深入伪 UPX 分析之一静态脱壳机

> 原文：[`www.pediy.com/kssd/pediy10/71605.html`](https://www.pediy.com/kssd/pediy10/71605.html)

废话不多说了，发出来给大家~~~~~~~~~

这个是代码算法解密程序，我自己重组了 `IAT` 和输入表，还有资源表。

刚出炉，估计问题多多~~~~~~

现在是完全可以把壳扒掉，估计修复的时候还有问题。。

`OD` 已经可以完全加载了。

不会手动脱壳的，直接把文件往上一脱就搞定了。。。。

我还在测试完善中。。。。

大家帮忙看哈~~~~~~~~~~~~~

![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬") ![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬") ![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")

这次不容易撒~~~~~~把算法代码整进了程序。由于修复还么找到合适的方法，所以是我根据我自己的研究而做的代码。。。![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!") ![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")

不好~~大家不要抱怨~~~

有时间就完善了~~~~

文件从这里下载，我附加了个简单的程序，做 `UPX` 测试用的~~~~

^\_^

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `UnPackGameGuard.rar` |

由于是算法直接解密，按道理来讲无所谓什么处理不处理~~~只要是 `UPX` 算法，就能搞定的。。。

试了几个都可以正常解密的，解密后的程序，有的可以用，有的用不了，但是 `OD`，全部能 `LOAD`。。。

正在研究解决中。。。

写的乱七八糟的~~~~

贴出来给大家笑话~~~

可能要做其它的事了，么机会再做了。把重要点的代码贴出来，希望给大家有帮助：

解密代码端的函数:

```
//-------------------------------------------------------------------------------------------------------------------------------

BOOL  DecryptCode()

{

  DWORD  dwEBP = 0;

  __asm

  {

    xor edi,edi

    je  _L2AFA

    jmp _LFEND

_L2AFA:

    mov edx,0x7D

    ;cmp     byte ptr [esp+0x08], 0x01  ;??临时注销

    mov esi,pvdUPX1Addr

    xor edi,edi

    je  _L2B1A

    jmp _LFEND

_L2B1A:

    mov ecx,dwUPX0Size

    ;lea edi,dword ptr[esi-ecx]

    mov edi,esi

    sub edi,ecx

    xor ecx,ecx

    je _L2B2A

    jmp _LFEND

_L2B2A:

    mov eax,0x7D

    cmp edx,eax

    not eax

/*-------------------------------------------------------

    jnz _L2B79

    mov eax,dword ptr fs:[0x30]

    test eax,eax

    js  _L2B67

    mov eax,dword ptr [eax + 0x0C]

    mov eax,dword ptr [eax + 0x0C]

    mov dword ptr[eax + 0x20],0x1000

    mov eax,dword ptr fs:[0x18]

    mov eax,dword ptr[eax+0x30]

    movzx eax,byte ptr[eax + 0x02]

    test eax,eax

    jnz _L2B78

    jmp _L2B79

_L2B67:

    xor eax,eax

    mov al,byte ptr fs:[0x20]

    test eax,eax

------------------------------------------------------------*/

    jnz  _L2B78

    jmp _L2B79

_L2B78:

_L2B79:

    or  dwEBP,0xFFFFFFFF

    jmp _L2B8A

    nop

_L2B80:

    mov al,byte ptr[esi]

    mov byte ptr[edi],al

    inc esi

    inc edi

_L2B86:

    add ebx,ebx

    jnz _L2B91

_L2B8A:

    mov ebx ,dword ptr[esi]

    sub esi, -4

    adc ebx,ebx

_L2B91:

    jb  _L2B80

    mov eax,0x01

_L2B98:

    add ebx,ebx

    jnz _L2BA3

    mov ebx,dword ptr[esi]

    sub esi , -4

    adc ebx,ebx

_L2BA3:

    adc eax,eax

    add ebx,ebx

    jnb _L2B98

    jnz _L2BB4

    mov ebx,dword ptr[esi]

    sub esi,-4

    adc ebx,ebx

    jnb _L2B98

_L2BB4:

    xor ecx,ecx

    sub eax,0x03

    jb  _L2BC8

    shl eax,0x08

    mov al,byte ptr[esi]

    inc esi

    xor eax , 0xFFFFFFFF

    je  _L2C3A

    mov dwEBP,eax

_L2BC8:

    add ebx,ebx

    jnz _L2BD3

    mov ebx,dword ptr[esi]

    sub esi , -4

    adc ebx,ebx

_L2BD3:

    adc ecx,ecx

    add ebx,ebx

    jnz _L2BE0

    mov ebx,dword ptr[esi]

    sub esi,-4

    adc ebx,ebx

_L2BE0:

    adc ecx,ecx

    jnz  _L2C04

    inc ecx

_L2BE5:

    add ebx,ebx

    jnz _L2BF0

    mov ebx,dword ptr[esi]

    sub esi , -4

    adc ebx,ebx

_L2BF0:

    adc ecx,ecx

    add ebx,ebx

    jnb _L2BE5

    jnz _L2C01

    mov ebx,dword ptr[esi]

      sub esi,-4

    adc ebx,ebx

    jnb _L2BE5

_L2C01:

    add ecx,0x02

_L2C04:

    cmp dwEBP,-0x0D00

    adc ecx,0x01

    mov edx,dwEBP

    lea edx ,dword ptr[edi + edx]

    cmp dwEBP,-4

    jbe _L2C24

_L2C15:

    mov al,byte ptr[edx]

    mov byte ptr[edi],al

    inc edx

    inc edi

    dec ecx

    jnz _L2C15

    jmp _L2B86

    nop

_L2C24:

    mov eax,dword ptr[edx]

      mov dword ptr[edi],eax

    add edx,0x04

    add edi,0x04

    sub ecx,0x04

    ja  _L2C24

    add edi,ecx

    jmp _L2B86

_L2C3A:

  }

  return TRUE;

  __asm

  {

_LFEND:

  }

  return FALSE;

}

//代码段解密后会有原始 PE 头，只需要加 4 个字节就是 PE 头，然后按 PE 头来处理。

//估计是版本不同 E8 后面的值随即改变的。

//-----------------------------------------------------------------------------------

BOOL  ModfiyJmpCall()

{

  LPBYTE  pbyte  = (LPBYTE)pvdUPX0Addr;

  DWORD dwC = 0x0637;

  for(int i = 0x00 ,n = 0 ; i< dwUPX0Size ; i++ )

  {

    if( (pbyte[i] - 0xE8) >= 0x00 && (pbyte[i] - 0xE8) <= 0x01 )

    {

      i++;

      if( pbyte[i] == 0x07 || pbyte[i] == 0x03 || pbyte[i] == 0x04 )

      {

        DWORD  dwBuf = *(LPDWORD)(&pbyte[i]);

        DWORD  dwA = HIWORD(dwBuf);

        //高低位互换:

        BYTE  byah = LOWORD(HIBYTE(dwA));

        BYTE  byal = LOWORD(LOBYTE(dwA));

        dwA  = byal *0x0100 + byah;

        dwA   -= (DWORD)&pbyte[i];

        dwA    += (DWORD)pvdUPX0Addr;

        *(LPDWORD)&pbyte[i] = dwA;

        txtout(_t("[%04d]调用修正[0x%08X]: 0x%08X <-> 0x%08X "),n,(DWORD)&pbyte[i],dwBuf,dwA);

        i += 0x03;

        n++;

      }

    }

  }

  txtout(_t("修正已经顺利结束,总计修正[%09d <-> 0x%08X]处."),n,n);

  return TRUE;

}

再说一个就是 `IAT` 表的格式为

模块名称相对地址 + `IAT` 表地址偏移 + 函数名称列表

函数名称前有个标志 `01` 代表是函数名称，`FF` 代表的是 `HINT` 号。

//------------------------------------------------------------------------------------------

00422C6E    8DBE 00000200   lea     edi, dword ptr [esi+20000]       ; 需要这个值计算 IAT 和输入表

00422C74    8B07           mov     eax, dword ptr [edi]             ; eax = [edi]

00422C76    09C0           or      eax, eax

00422C78    74 45          je      short 00422CBF                   ; eax = 0 -> jmp

00422C7A    8B5F 04        mov     ebx, dword ptr [edi+4]

00422C7D    8D8430 602B0200 lea    eax, dword ptr [eax+esi+22B60]    ; 需要这个的。

//-------------------------------------------------------------------------------------------
```

大家有兴趣的可以写了，`UPX` 不算很难的。

我可能需要做其他的事，估计么时间做了。关键的地方都贴出来了。

应该大家都能写出来了。
