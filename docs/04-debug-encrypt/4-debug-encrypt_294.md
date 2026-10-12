
# 统计分析攻击实例

> 原文：[`www.pediy.com/kssd/pediy10/90569.html`](https://www.pediy.com/kssd/pediy10/90569.html)

统计分析攻击实例

by `stalker`

引用: 统计分析攻击就是指密码分析者根据明文、密文和密钥的统计规律来破译密码的方法。例如，在经典的换位密码、置换密码体制中，可通过分析单字母、双字母、三字母等的频率和其他统计参数而破译。对抗统计分析攻击的主要方法是设法使明文的统计特性不带入密文。这样，密文不带有明文的痕迹，而呈现出极大的随机性，从而挫败统计分析攻击。能够对抗统计分析攻击已成为近代密码的基本要求。

刘嘉勇等.《应用密码学》北京:清华大学出版社,2008

## 攻击目标

`target.rar`

## 目标保护方法

代码与数据结合，把用户输入作为 `key` 对已加密的代码进行解码，如果 `key` 正确，则解出正确代码，提示成功，否则解出错误代码，程序由 `ErrorHandler` 结束或者直接崩掉。(我最先是在论坛出版的《软件加密技术内幕》一书中学到这一方法)

## 关键代码如下

```
0040103A    .   B9 70000000   mov     ecx, 70
0040103F    .   BE 4C104000   mov     esi, 0040104C
00401044    >   3006          xor     byte ptr [esi], al
00401046    .   49            dec     ecx
00401047    .   E3 03         jecxz   short 0040104C
00401049    .   46            inc     esi
0040104A    .^  EB F8         jmp     short 00401044
0040104C    >   DC37          fdiv    qword ptr [edi]
0040104E    .   71 79         jno     short 004010C9
00401050    .   57            push    edi
00401051    .   56            push    esi
00401052    .   67:6F         outs    dx, dword ptr es:[di]
00401054    .   6E            outs    dx, byte ptr es:[edi]
00401055    .   66:5F         pop     di
00401057    .   52            push    edx
00401058    .   16            push    ss
00401059    .   37            aaa
0040105A    .   37            aaa
0040105B    .   5F            pop     edi
0040105C    .   17            pop     ss
0040105D    .   73 58         jnb     short 004010B7
0040105F    .   59            pop     ecx
00401060    .   5F            pop     edi
00401061    .   60            pushad
00401062    .   52            push    edx
00401063    .   5B            pop     ebx
00401064    .   5B            pop     ebx
00401065    .   BC F35F4416   mov     esp, 16445FF3
0040106A    .   37            aaa
0040106B    .   37            aaa
0040106C    .   5F            pop     edi
0040106D    .   52            push    edx
0040106E    .   17            pop     ss
0040106F    .   47            inc     edi
00401070    .   5B            pop     ebx
00401071    .   5F            pop     edi
00401072    .   17            pop     ss
00401073    .   53            push    ebx
00401074    .   58            pop     eax
00401075    .   59            pop     ecx
00401076    .   5F            pop     edi
00401077    .   42            inc     edx
00401078    .   1041 52       adc     byte ptr [ecx+52], al
0040107B    .   5F            pop     edi
0040107C    .   59            pop     ecx
0040107D    .   17            pop     ss
0040107E    .   4E            dec     esi
0040107F    .   58            pop     eax
00401080    .   5F            pop     edi
00401081    .   50            push    eax
00401082    .   42            inc     edx
00401083    .   5C            pop     esp
00401084    .   56            push    esi
00401085    .   5F            pop     edi
00401086    .   5B            pop     ebx
00401087    .   17            pop     ss
00401088    .   53            push    ebx
00401089    .   58            pop     eax
0040108A    .   5F            pop     edi
0040108B    .   16            push    ss
0040108C    .   43            inc     ebx
0040108D    .   52            push    edx
0040108E    .   5B            pop     ebx
0040108F    .   5F            pop     edi
00401090    .   5A            pop     edx
00401091    .   56            push    esi
00401092    .   45            inc     ebp
00401093    .   43            inc     ebx
00401094    .   5F            pop     edi
00401095    .   44            inc     esp
00401096    .   58            pop     eax
00401097    .   17            pop     ss
00401098    .   44            inc     esp
00401099    .   5F            pop     edi
0040109A    .   56            push    esi
0040109B    .   45            inc     ebp
0040109C    .   52            push    edx
0040109D    .   17            pop     ss
0040109E    .   5F            pop     edi
0040109F    .   4E            dec     esi
004010A0    .   58            pop     eax
004010A1    .   42            inc     edx
004010A2    .   17            pop     ss
004010A3    .   BC EB5D7767   mov     esp, 67775DEB
004010A8    .   64:5D         pop     ebp
004010AA    .   37            aaa
004010AB    .   C8 227B17     enter   7B22, 17
004010AF    .   77 37         ja      short 004010E8
004010B1    .   B4 F3         mov     ah, 0F3
004010B3    .   0BDC          or      ebx, esp
004010B5    .   37            aaa
004010B6    .   57            push    edi
004010B7    >   56            push    esi
004010B8    .   51            push    ecx
004010B9    .   6E            outs    dx, byte ptr es:[edi]
004010BA    .   51            push    ecx
004010BB    .   66            db      66                              ;  CHAR 'f'
```

## 原理概述

从上面看到，从 `40104C` 到 `4010BB` 范围内的代码已经被加密，从 `xor byte ptr [esi], al` 这一句代码可以看出，程序采用了简单的 `xor` 加密，并且 `key` 的长度为 `1 byte`。现在我们只拥有密文（被加密后的代码），如何能获取明文呢（加密前的代码）?因为 `key` 的长度仅仅为 `1 byte`，即使使用最笨拙的穷举攻击法，我们也能很快找出正确的 `key`。但是这次我们要使用的是统计分析法，大家需要知道程序代码的一个通用的统计特性------在程序代码中通常 `0` 最多。

由于 `0 xor key = key`

因此密文中与之对应的特性就是------在密文（被加密的程序代码）中通常 `key` 最多。

现在我们要做的就是统计 `40104E` 到 `4010BD` 范围内各个数据(`0x00`->`0xFF`)出现的次数，然后从出现频率最大的数据开始尝试对密文进行解码。

利用 `WinHex` 从 `crackme.exe` 中提取出被加密的代码，并用如下程序对其进行统计

```
#include <stdio.h>

int main(void)
{ 
    int i,j;
    unsigned char m[112] = {
      0xDC, 0x37, 0x71, 0x79, 0x57, 0x56, 0x67, 0x6F, 0x6E, 0x66, 0x5F, 0x52, 0x16, 0x37, 0x37, 0x5F, 
      0x17, 0x73, 0x58, 0x59, 0x5F, 0x60, 0x52, 0x5B, 0x5B, 0xBC, 0xF3, 0x5F, 0x44, 0x16, 0x37, 0x37, 
      0x5F, 0x52, 0x17, 0x47, 0x5B, 0x5F, 0x17, 0x53, 0x58, 0x59, 0x5F, 0x42, 0x10, 0x41, 0x52, 0x5F, 
      0x59, 0x17, 0x4E, 0x58, 0x5F, 0x50, 0x42, 0x5C, 0x56, 0x5F, 0x5B, 0x17, 0x53, 0x58, 0x5F, 0x16, 
      0x43, 0x52, 0x5B, 0x5F, 0x5A, 0x56, 0x45, 0x43, 0x5F, 0x44, 0x58, 0x17, 0x44, 0x5F, 0x56, 0x45, 
      0x52, 0x17, 0x5F, 0x4E, 0x58, 0x42, 0x17, 0xBC, 0xEB, 0x5D, 0x77, 0x67, 0x64, 0x5D, 0x37, 0xC8, 
      0x22, 0x7B, 0x17, 0x77, 0x37, 0xB4, 0xF3, 0x0B, 0xDC, 0x37, 0x57, 0x56, 0x51, 0x6E, 0x51, 0x66
      },s[256] = {0};
      //数组 m 即为从 WinHex 中提取出来的密文，数组 s 用于保存每个数据出现的次数
      for(i = 0;i < 112;i++)
          for(j = 0;j <= 255;j++)
              if(m[i] == j)
                  s[j]++;
      for(i = 0x00;i <= 0xFF;i++)
          printf("0x%02X:%d\n",i,s[i]);
      system("pause");
      return 0;
}
```

从输出结果来看，以下几个数据的出现频率较大

```
0x5F:15
0x17:9
0x37:8
0x52:6
0x58:6
0x56:5
0x5B:5
```

从 `0x5F` 开始做 `key` 尝试对密文进行解密，最终确定 `0x37` 即为正确的 `key`。

在这一个例子中，我们只要对算法稍做改进，例如，每次 `xor` 之后对 `key` 做一定的变换，就可以使明文的统计特性不被带入密文，从而抵抗统计分析法。

## PS

我是一只菜鸟，刚刚开始看最开始提到的那本书，才发现以前知道的这么一点点东西居然还有科学的名字

感谢经验丰富的程序员天杀，程序代码中通常 `0` 最多这一特性是很久以前我和他聊天的时候他告诉我的。
