
# [原创]PhotoZoomPro2 2.1.8 脱壳修复与写代码破解(合精版)

【文章标题】: PhotoZoomPro2 2.1.8 脱壳修复与破解(合精版)

【文章作者】: [qyc]小Q

【作者主页】: 9bdK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3W2$3x3W2)9J5k6h3y4F1

【软件名称】: PhotoZoomPro2

【软件大小】: 3.35M

【下载地址】: 自己搜索下载

【加壳方式】: ASProtect 2.1x SKE -> Alexey Solodovnikov-->Version: ASProtect 1.35 build 04.25 or 06.26 Release [Extract]

【保护方式】: 加密壳

【编写语言】: VC++

【使用工具】: PEID IRE OD

【操作平台】: Win XP SP2

【软件介绍】: PhotoZoomPro2是一个图片放大软件

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 【详细过程】

原贴 : http://bbs.pediy.com/showthread.php?threadid=37499  这个贴只是修正! 让文章完整一些!

PhotoZoomPro2是一个图片放大软件,小Q一直以来都是用最低版本的!知到有最新版!可以就是没有用的机会,因为没有破解!

自己以前脱壳技术水平又不高一直没对它下手! 直到Aspr2.XX_IATfixer_v2.2s.osc的出现,才给了偶这次动它的机会,今天

公司叫偶放大图片就再次下PhotoZoomPro2 最新版本是: 2.1.8

PEID--->ASProtect 2.1x SKE -> Alexey Solodovnikov

插件--->Version: ASProtect 1.35 build 04.25 or 06.26 Release [Extract]

1. 解决问题: 变态自校验

OD 加载 程序 Aspr2.XX_IATfixer_v2.2s.osc 脚本完全dumped,IRE修复IAT,一个指针未修复,用插件搞定! 下面是精彩部分

运行程序发现出错不能运行,一开始还以为壳偷了代码,认为10分变态,我所知ASProtect 1.35是不会偷代码的!

脱壳后变态入口:

```
00584FEF > $  E8 190D0100   call    00595D0D-------->F8死// F7进
00584FF4   .^ E9 16FEFFFF   jmp     00584E0F
00584FF9   $  6A 0C         push    0C
00584FFB   .  68 F0D16F00   push    006FD1F0
00585000   .  E8 DFC40000   call    005914E4
00585005   .  8B75 08       mov     esi, [ebp+8]
```

来到:

```
00584F47   .  85C0          test    eax, eax
00584F49   .  7D 08         jge     short 00584F53
00584F4B   .  6A 09         push    9
00584F4D   .  E8 90330000   call    005882E2
00584F52   .  59            pop     ecx
00584F53   >  53            push    ebx
00584F54   .  E8 A5340000   call    005883FE--------------->内存异常!程序不可读! F7进
00584F59   .  59            pop     ecx
00584F5A   .  85C0          test    eax, eax
00584F5C   .  74 07         je      short 00584F65
```

下面就是变态过程:

```
00588456  |. /73 0F         jnb     short 00588467
00588458  |> |8B06          /mov     eax, [esi]
0058845A  |. |85C0          |test    eax, eax
0058845C  |. |74 02         |je      short 00588460
0058845E  |. |FFD0          |call    eax    --------> 相当变态的动态地址(异常将在这里发生)!
00588460  |> |83C6 04       |add     esi, 4
00588463  |. |3BF7          |cmp     esi, edi
00588465  |.^|72 F1         \jb      short 00588458------->来回不知多少次,你别用F8了!
00588467  |> \833D BC1F7500>cmp     dword ptr [751FBC], 0
```

直接 F9 后,错误提示内存00FC0000不可读.见图! 好伽伙,这招毒啊!

![](http://bbs.pediy.com/upload/2006/4/image/bug.jpg_540.jpg)

点确定后! 看堆栈: (这是由CCDebuger的提示!,错误精确定位法)

```
0012FF14   005BFC10  返回到 dumped\_.005BFC10 来自 dumped\_.0058CD38---->这里回车
0012FF18   005EB69F  返回到 dumped\_.005EB69F 来自 dumped\_.005BFC06
0012FF1C   00632E7C  ASCII "file" -------------------->这分明就是在检验文件吗!
```

```
005BFC06  /$  B8 A3125E00   mov     eax, 005E12A3
005BFC0B  |.  E8 28D1FCFF   call    0058CD38--------->这里一定是出错地方 (F2下断)
005BFC10  |.  51            push    ecx-------------->找到这里
005BFC11  |.  56            push    esi
005BFC12  |.  8BF1          mov     esi, ecx
```

重载进入call    0058CD38 :

```
0058CD38  /$  68 0000FC00   push    0FC0000------>是不是错误提示地址啊
0058CD3D  \.  C3            retn
0058CD3E      AE            db      AE
0058CD3F      2D            db      2D                               ;  CHAR '-'
0058CD40      1B            db      1B
0058CD41      77            db      77                               ;  CHAR 'w'
0058CD42      BD            db      BD
0058CD43      5E            db      5E                               ;  CHAR '^'
0058CD44      8B            db      8B
0058CD45      BC            db      BC
0058CD46      19            db      19
0058CD47      42            db      42                               ;  CHAR 'B'
0058CD48      B5            db      B5
0058CD49      35            db      35                               ;  CHAR '5'
0058CD4A      E4            db      E4
0058CD4B      AF            db      AF
0058CD4C      57            db      57                               ;  CHAR 'W'
0058CD4D      E7            db      E7
0058CD4E      C6            db      C6
0058CD4F      2C            db      2C                               ;  CHAR ','
0058CD50      2E            db      2E                               ;  CHAR '.'
0058CD51      22            db      22                               ;  CHAR '"'
0058CD52      D5            db      D5
0058CD53      0A            db      0A
0058CD54      D9            db      D9
0058CD55      BD            db      BD
0058CD56      CA            db      CA
0058CD57      CC            int3
```

执行push 0FC0000后就是空白代码,程序出错,程序被WINDOWS 关闭,所有问题连接而来(狂晕中).

55555,要是偶能用程序语言写出代码就好罗,可是写不出,呵呵!

以上所知!找到了问题所在,那就是出注意解决问题啦! 各位考考你们,让想到啦,请举手哈(鸡蛋青菜全向偶飞来!不要,我马上解决)

解决问题很简单:跟踪原程序(哈哈!大家别扁偶啊,老大们都是这样做的)

以下代码是由cyto兄提供的(代码大小字节写入刚刚好) (不过用我跟到的代码也行,不过你得找空间写,详情请见原贴)

Emulate Standard system Functions.

可以在原处还原.

```
0058CD38     6A FF              push -1
0058CD3A     50                 push eax
0058CD3B     64:A1 00000000     mov eax,dword ptr fs:[0]
0058CD41     50                 push eax
0058CD42     8B4424 0C          mov eax,dword ptr ss:[esp+C]
0058CD46     64:8925 00000000   mov dword ptr fs:[0],esp
0058CD4D     896C24 0C          mov dword ptr ss:[esp+C],ebp
0058CD51     8D6C24 0C          lea ebp,dword ptr ss:[esp+C]
0058CD55     50                 push eax
0058CD56     C3                 retn
```

16进:

```
6A FF 50 64 A1 00 00 00 00 50 8B 44 24 0C 64 89 25 00 00 00 00 89 6C 24 0C 8D 6C 24 0C 50 C3
```

OK ,保存后,测试运行程序(已可运行!)

以下代码machenglin兄提供修改(不过,程序已可运行),得问一下machenglin兄为何要修改???

第1处修改:

原来是:

```
00400138    0010              add byte ptr ds:[eax],dl
0040013A    0000              add byte ptr ds:[eax],al
0040013C    0002              add byte ptr ds:[eax],dl
```

修改为:

```
00400138    0010              add byte ptr ds:[eax],dl
0040013A    0000              add byte ptr ds:[eax],al
0040013C    0010              add byte ptr ds:[eax],dl
```

第2处修改:

原来是:

```
00720078    0000              add byte ptr ds:[eax],al
0072007A    0000              add byte ptr ds:[eax],al
```
