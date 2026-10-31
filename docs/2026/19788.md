
# [原创]看雪CTF2017 第五题分析

`od`无法调试，先使用`ida`静态分析，发现该程序加载驱动，`patch`掉驱动后再动态调试。

在`DeviceIoControl`和`CreateFileA`处下断，查看调用栈，来到004013E0函数

如下地址
```
00401580   .  6A 00         push    0

00401582   .  6A 12         push    12

00401584   .  51            push    ecx

00401585   .  FFD3          call    ebx

00401587   >  8D5424 1C     lea     edx, dword ptr [esp+1C]

0040158B   .  8BCD          mov     ecx, ebp

0040158D      52            push    edx

0040158E      68 0CD14200   push    0042D10C                         ;  ASCII "vmxdrv"

00401593      E8 08050000   call    00401AA0                         ;  startup service

00401598   .  85C0          test    eax, eax

0040159A   .  8945 64       mov     dword ptr [ebp+64], eax

0040159D   .  75 26         jnz     short 004015C5

0040159F   .  68 0CD14200   push    0042D10C                         ;  ASCII "vmxdrv"
```

将 0040158D - 00401593 `NOP`掉，过掉驱动，注意：当运行到此处时将`exe`所在目录下的`vmxdrv.sys`改个名，后面我们用`IDA`分析。

往下走来到
```
004015C5   > \8D4C24 1C     lea     ecx, dword ptr [esp+1C]

004015C9   .  51            push    ecx                              ; /FileName

004015CA   .  FF15 A8324200 call    dword ptr [<&KERNEL32.DeleteFile>; \DeleteFileA

004015D0      6A 05         push    5

004015D2      8BCD          mov     ecx, ebp

004015D4      E8 770C0000   call    00402250 《- 启动线程与驱动交互

004015D9   .  8B8C24 200100>mov     ecx, dword ptr [esp+120]

004015E0   .  5F            pop     edi

004015E1   .  5E            pop     esi
```

将 004015D0 - 004015D4 `NOP`掉，过掉线程检查

在`GetWindowTextA`处下断，运行程序输入1111111111111，回车断下，查看调用栈来到
```
0040179C   .  57            push    edi

0040179D   .  E8 A1650100   call    00417D43                         ;  get_sn

004017A2   .  8D4C24 0C     lea     ecx, dword ptr [esp+C]

004017A6   .  C64424 24 01  mov     byte ptr [esp+24], 1

004017AB   .  E8 4A6B0100   call    004182FA

004017B0   .  8D4C24 0C     lea     ecx, dword ptr [esp+C]

004017B4   .  E8 536B0100   call    0041830C

004017B9   .  E8 BDD60100   call    0041EE7B

004017BE   .  8B4C24 0C     mov     ecx, dword ptr [esp+C]

004017C2   .  8B40 04       mov     eax, dword ptr [eax+4]

004017C5   .  8379 F8 06    cmp     dword ptr [ecx-8], 6             ;  len(sn) == 6

004017C9      0F85 F3000000 jnz     004018C2
```

确定`sn`长度为6，继续往下分析
```
004017F1   .  6A 00         push    0

004017F3   .  E8 6B6A0100   call    00418263

004017F8   .  50            push    eax

004017F9   .  8BCE          mov     ecx, esi

004017FB   .  E8 50050000   call    00401D50                         ;  do file <-> sys

00401800   .  EB 10         jmp     short 00401812
```

将`sn`反向，然后通过00401D50函数传到驱动进行处理
```
00401827   .  8BCE          mov     ecx, esi

00401829   .  E8 F2000000   call    00401920                         ;  md5??

0040182E   .  6A 0A         push    0A

00401830   .  8D4424 18     lea     eax, dword ptr [esp+18]

00401834   .  6A 02         push    2

00401836   .  50            push    eax

00401837   .  8D4C24 1C     lea     ecx, dword ptr [esp+1C]

0040183B   .  E8 38420100   call    00415A78                         ;  sub_str

00401840   .  8D4C24 14     lea     ecx, dword ptr [esp+14]
```

对驱动传出来的数据进行`md5`，在线`md5`验证一下可确定

获取`md5`后的字符串2-12，共10个字符的子串
```
00401872   .  8B4C24 14     mov     ecx, dword ptr [esp+14]

00401876   .  68 6CD14200   push    0042D16C                         ;  ASCII "888aeda4ab"

0040187B   .  51            push    ecx

0040187C   .  E8 309F0000   call    0040B7B1                         ;  _mbsicmp

00401881   .  83C4 08       add     esp, 8

00401884   .  85C0          test    eax, eax

00401886   .  75 09         jnz     short 00401891
```

结果与"888aeda4ab"比较

下面使用`ida`分析驱动，驱动很小，只分析一下主要功能
```
  if ( result <= 16 )

  {

    memcpy(&v6, a1, result);

    v4 = 0;

    if ( dword_114D8 )

      ++v6;

    if ( v3 > 0 )

    {

      do

      {

        *(&v6 + v4) += v4;

        ++v4;

      }

      while ( v4 < v3 );

    }

    md5_init((int)&v5);

    md5__2((int)&v5, &v6, strlen(&v6));

    result = md5_final((unsigned int *)&v5, a2);

  }
```

首先，对输入值进行简单加法，然后进行`md5`运算，再然后将结果传回给`exe`。

通过分析，下面使用`python`进行穷举，代码如下：
```
import hashlib

import sys



def hash_md5(src):

    myMd5 = hashlib.md5()

    myMd5.update(src)

    myMd5_Digest = myMd5.hexdigest()

    return myMd5_Digest



def is_ok(v):

    if v[2:12] == "888aeda4ab":

        return 1    return 0def do_md5(src):

    x = ""    x += chr(ord(src[0])+1)

    for i in range(1, len(src)):

        x += chr(ord(src[i])+i)

    x = hash_md5(hash_md5(x))

    return x



def get_sn(str, num):

    if (num == 1):

        for x in str:

            yield x

    else:

        for x in str:

            for y in get_sn(str, num - 1):

                yield x + y





if __name__== '__main__':

    print is_ok('a3888aeda4abba91f31c8e0caae48cb9')



    # 000000    x =  do_md5('000000')

    print x[2:12] == "fd9e2ddbd6"    for sn in get_sn('0123456789abcdefghijklmnopqrstuvwxyz', 6):

        x = do_md5(sn)

        if sn[2:6]=='0000':

            print sn

        if is_ok(x) == 1:

            print 'sn='+sn

            break
```

输出：`sn`=6891us，需要反向输入即：su1986

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
