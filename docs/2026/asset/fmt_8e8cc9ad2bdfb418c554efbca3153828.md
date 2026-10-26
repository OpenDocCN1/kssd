
# [原创]AP PDF Password Recover，破解加注册机

- 【文章标题】: AP PDF Password Recover，破解加注册机
- 【文章作者】: redblack
- 【作者邮箱】: redblack@zju.edu.cn
- 【软件名称】: AP PDF Password Recover
- 【下载地址】: `9d9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8G2N6$3&6D9L8$3q4V1i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4G2N6i4u0U0k6g2)9J5c8U0p5^5x3U0M7J5x3o6j5`
- 【加壳方式】: UltralProtect 1.x
- 【编写语言】: VC
- 【使用工具】: `peid`, `OD`, `MasmPlus`
- 【操作平台】: XP sp3

注：灰常简单的注册机，老鸟飘过。。。。。仅供入门的兄弟们学习使用![](https://bbs.kanxue.com/view/img/face/50.gif)![](https://bbs.kanxue.com/view/img/face/50.gif)

昨晚下载了一个带密码的pdf，我知道密码，可是每次都要输入很麻烦，于是上网找到了一个叫做`AP PDF Password Recovery.exe`的软件，无奈，这家伙是要注册的。抄家伙上啦！

## 查壳

首先查壳：UltraProtect 1.x -> RISCO Software Inc.

我印象中，这个壳会有`int3`的反调试。于是忽略`int3`异常。

一开始是想脱掉它，然后再破解的。后来一个偶然的机会让我发现这个是没有必要滴

运行之后，输入假码，报`Series number error,please check it and try again`

接下来大家就知道怎么做了吧(不过这里稍微要麻烦一点)

## 破解流程

破解流程如下：

首先 `od` 载入，下 `bp MessageBoxA`

```
00519000 > $  60            pushad                                   ；这里是入口点
00519001   .  72 03         jb      short 00519006
00519003   .  73 01         jnb     short 00519006
00519005      71            db      71                               ;  CHAR 'q'
00519006   .  7E 03         jle     short 0051900B
00519008   >  7F 01         jg      short 0051900B
0051900A   .^ EB FC         jmp     short 00519008
0051900C      66            db      66                               ;  CHAR 'f'
0051900D      81            db      81
0051900E   .- E1 A7         loopde  short 00518FB7
```

这样的话 我们F9运行，窗口顺利出线，输入假码后，并没有断注。。。。无限郁闷。。。

很明显，壳有检测`CC`断点的功能，我们希望他没有检测硬件断点（实际情况他没有清除）

Alt+B 打开断点窗口，在刚才的`MessagBoxA`断点上面右键-->反汇编窗口中跟随

F2取消`CC`断点，我们右键下硬件执行断点

重新运行程序。F9，输入假码，顺利断下，执行到返回。往上翻，我们看到如下的代码

```
00409520  /.  55            push    ebp
00409521  |.  8BEC          mov     ebp, esp
00409523  |.  83C4 C4       add     esp, -3C
00409526  |.  53            push    ebx
00409527  |.  56            push    esi
00409528  |.  57            push    edi
00409529  |.  8BD8          mov     ebx, eax                     ；这里有个8 L的特征码
0040952B  |.  BE E41C4C00   mov     esi, 004C1CE4
00409530  |.  8D7D C4       lea     edi, [ebp-3C]
00409533  |.  B8 A81F4C00   mov     eax, 004C1FA8
00409538  |.  E8 878A0A00   call    004B1FC4
0040953D  |.  66:C747 10 14>mov     word ptr [edi+10], 14
00409543  |.  33D2          xor     edx, edx
00409545  |.  8955 FC       mov     [ebp-4], edx
00409548  |.  8D55 FC       lea     edx, [ebp-4]
0040954B  |.  FF47 1C       inc     dword ptr [edi+1C]
0040954E  |.  8B83 F4020000 mov     eax, [ebx+2F4]
00409554  |.  E8 53C10700   call    004856AC
00409559  |.  66:C747 10 08>mov     word ptr [edi+10], 8
0040955F  |.  837D FC 00    cmp     dword ptr [ebp-4], 0        ；这里判断密码是否为空
00409563  |.  74 05         je      short 0040956A
00409565  |.  8B4D FC       mov     ecx, [ebp-4]                ；当当当当当，我们输入密码出现
00409568  |.  EB 03         jmp     short 0040956D
0040956A  |>  8D4E 47       lea     ecx, [esi+47]
0040956D  |>  51            push    ecx                              ; /Arg2
0040956E  |.  53            push    ebx                              ; 将8 L和我们的密码传入
                                                                    ；关键call，我们跟入
0040956F  |.  E8 50FFFFFF   call    004094C4                         ; 很明显，这个是关键call
00409574  |.  83C4 08       add     esp, 8
00409577  |.  3C 01         cmp     al, 1
00409579  |.  0F85 45010000 jnz     004096C4                         ；这里跳转到下面的badboy，上面正好有比较，我们先不让他跳，发现成功注册。
0040957F  |.  6A 40         push    40                               ；有同学就要说，直接爆破就好了嘛，其实这个是不行的，因为。。因为，他加壳了。
                                                                    ；我们现在改了，如果写入源文件的话，他再解压后就不能运行了。我们只能进行追码
                                                                    ；于是就产生了俺的第一个汇编的注册机，oh yeah
00409581  |.  8D86 A8000000 lea     eax, [esi+A8]
00409587  |.  50            push    eax
00409588  |.  8D56 48       lea     edx, [esi+48]
0040958B  |.  52            push    edx
0040958C  |.  8BC3          mov     eax, ebx
0040958E  |.  E8 2D280800   call    0048BDC0
00409593  |.  50            push    eax
00409594  |.  E8 DB560B00   call    004BEC74
00409599  |.  66:C747 10 2C>mov     word ptr [edi+10], 2C
0040959F  |.  33D2          xor     edx, edx
004095A1  |.  8955 F4       mov     [ebp-C], edx
004095A4  |.  8D55 F4       lea     edx, [ebp-C]
004095A7  |.  FF47 1C       inc     dword ptr [edi+1C]
004095AA  |.  A1 44A24C00   mov     eax, [4CA244]
004095AF  |.  8B00          mov     eax, [eax]
004095B1  |.  E8 BEE60600   call    00477C74
004095B6  |.  8D45 F4       lea     eax, [ebp-C]
004095B9  |.  8B00          mov     eax, [eax]
004095BB  |.  33D2          xor     edx, edx
004095BD  |.  8955 F0       mov     [ebp-10], edx
004095C0  |.  8D55 F0       lea     edx, [ebp-10]
004095C3  |.  FF47 1C       inc     dword ptr [edi+1C]
004095C6  |.  E8 E9840200   call    00431AB4
004095CB  |.  8D45 F0       lea     eax, [ebp-10]
004095CE  |.  50            push    eax
004095CF  |.  8D96 BB000000 lea     edx, [esi+BB]
004095D5  |.  8D45 EC       lea     eax, [ebp-14]
004095D8  |.  E8 6F440B00   call    004BDA4C
004095DD  |.  FF47 1C       inc     dword ptr [edi+1C]
004095E0  |.  33C0          xor     eax, eax
004095E2  |.  8945 F8       mov     [ebp-8], eax
004095E5  |.  8D55 EC       lea     edx, [ebp-14]
004095E8  |.  FF47 1C       inc     dword ptr [edi+1C]
004095EB  |.  8D4D F8       lea     ecx, [ebp-8]
004095EE  |.  58            pop     eax
004095EF  |.  E8 54460B00   call    004BDC48
004095F4  |.  FF4F 1C       dec     dword ptr [edi+1C]
004095F7  |.  8D45 EC       lea     eax, [ebp-14]
004095FA  |.  BA 02000000   mov     edx, 2
004095FF  |.  E8 EC450B00   call    004BDBF0
00409604  |.  FF4F 1C       dec     dword ptr [edi+1C]
00409607  |.  8D45 F0       lea     eax, [ebp-10]
0040960A  |.  BA 02000000   mov     edx, 2
0040960F  |.  E8 DC450B00   call    004BDBF0
00409614  |.  FF4F 1C       dec     dword ptr [edi+1C]
00409617  |.  8D45 F4       lea     eax, [ebp-C]
0040961A  |.  BA 02000000   mov     edx, 2
0040961F  |.  E8 CC450B00   call    004BDBF0
00409624  |.  66:C747 10 20>mov     word ptr [edi+10], 20
0040962A  |.  837D F8 00    cmp     dword ptr [ebp-8], 0
0040962E  |.  74 05         je      short 00409635
00409630  |.  8B4D F8       mov     ecx, [ebp-8]
00409633  |.  EB 06         jmp     short 0040963B
00409635  |>  8D8E D6000000 lea     ecx, [esi+D6]
0040963B  |>  51            push    ecx
0040963C  |.  837D FC 00    cmp     dword ptr [ebp-4], 0
00409640  |.  74 05         je      short 00409647
00409642  |.  8B45 FC       mov     eax, [ebp-4]
00409645  |.  EB 06         jmp     short 0040964D
00409647  |>  8D86 D5000000 lea     eax, [esi+D5]
0040964D  |>  50            push    eax
0040964E  |.  8D96 CE000000 lea     edx, [esi+CE]
00409654  |.  52            push    edx
00409655  |.  8D8E C7000000 lea     ecx, [esi+C7]
0040965B  |.  51            push    ecx
0040965C  |.  E8 2D500B00   call    004BE68E
00409661  |.  33D2          xor     edx, edx
00409663  |.  8B83 08030000 mov     eax, [ebx+308]
00409669  |.  8B08          mov     ecx, [eax]
0040966B  |.  FF51 64       call    [ecx+64]
0040966E  |.  66:C747 10 38>mov     word ptr [edi+10], 38
00409674  |.  8D96 D7000000 lea     edx, [esi+D7]
0040967A  |.  8D45 E8       lea     eax, [ebp-18]
0040967D  |.  E8 CA430B00   call    004BDA4C
00409682  |.  FF47 1C       inc     dword ptr [edi+1C]
00409685  |.  8B10          mov     edx, [eax]
00409687  |.  8B83 00030000 mov     eax, [ebx+300]
0040968D  |.  E8 4AC00700   call    004856DC
00409692  |.  FF4F 1C       dec     dword ptr [edi+1C]
00409695  |.  8D45 E8       lea     eax, [ebp-18]
00409698  |.  BA 02000000   mov     edx, 2
0040969D  |.  E8 4E450B00   call    004BDBF0
004096A2  |.  C783 4C020000>mov     dword ptr [ebx+24C], 1
004096AC  |.  FF4F 1C       dec     dword ptr [edi+1C]
004096AF  |.  8D45 F8       lea     eax, [ebp-8]
```
