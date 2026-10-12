
# AP PDF Password Recover 破解加注册机

> 原文：[`www.pediy.com/kssd/pediy11/121877.html`](https://www.pediy.com/kssd/pediy11/121877.html)

**文章标题**：AP PDF Password Recover，破解加注册机  
**文章作者**：`redblack`  
**作者邮箱**：`redblack@zju.edu.cn`  
**软件名称**：AP PDF Password Recover  
**下载地址**：`http://download.csdn.net/source/1827206`  
**加壳方式**：UltralProtect 1.x  
**编写语言**：VC  
**使用工具**：`peid`, `OD`, `MasmPlus`  
**操作平台**：XP sp3

注：灰常简单的注册机，老鸟飘过。。。。。仅供入门的兄弟们学习使用![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")

昨晚下载了一个带密码的 pdf，我知道密码，可是每次都要输入很麻烦，于是上网找到了一个叫做

`AP PDF Password Recovery.exe` 的软件，无奈，这家伙是要注册的。抄家伙上啦！

首先查壳：`UltraProtect 1.x` -> `RISCO Software Inc.`

我印象中，这个壳会有 `int3` 的反调试。于是忽略 `int3` 异常。

一开始是想脱掉它，然后再破解的。后来一个偶然的机会让我发现这个是没有必要滴

运行之后，输入假码，报 `Series number error, please check it and try again`

接下来大家就知道怎么做了吧(不过这里稍微要麻烦一点)

## 破解流程

首先 `od` 载入，下 `bp MessageBoxA`

```
00519000  >  $   60            pushad                         ；这里是入口点
00519001   .  72 03         jb      short 00519006
00519003   .  73 01         jnb     short 00519006
00519005     71            db      71                       ;  CHAR 'q'
00519006   .  7E 03        jle     short 0051900B
00519008   >  7F 01        jg      short 0051900B
0051900A   .^ EB FC       jmp     short 00519008
0051900C     66            db      66                       ;  CHAR 'f'
0051900D     81            db      81
0051900E   .- E1 A7       loopde  short 00518FB7
```

这样的话 我们 `F9` 运行，窗口顺利出线，输入假码后，并没有断注。。。。无限郁闷。。。

很明显，壳有检测 `CC` 断点的功能，我们希望他没有检测硬件断点（实际情况他没有清除）

`Alt+B` 打开断点窗口，在刚才的 `MessageBoxA` 断点上面右键-->反汇编窗口中跟随

`F2` 取消 `CC` 断点，我们右键下硬件执行断点

重新运行程序。`F9`，输入假码，顺利断下，执行到返回。往上翻，我们看到如下的代码

```
00409520  /.   55            push    ebp
00409521  |.   8BEC          mov     ebp, esp
00409523  |.   83C4 C4       add     esp, -3C
00409526  |.   53            push    ebx
00409527  |.   56            push    esi
00409528  |.   57            push    edi
00409529  |.   8BD8          mov     ebx, eax                 ；这里有个 8 L 的特征码
0040952B  |.   BE E41C4C00   mov     esi, 004C1CE4
00409530  |.   8D7D C4       lea     edi, [ebp-3C]
00409533  |.   B8 A81F4C00   mov     eax, 004C1FA8
00409538  |.   E8 878A0A00   call    004B1FC4
0040953D  |.   66:C747 10 14>mov     word ptr [edi+10], 14
00409543  |.   33D2          xor     edx, edx
00409545  |.   8955 FC       mov     [ebp-4], edx
00409548  |.   8D55 FC       lea     edx, [ebp-4]
0040954B  |.   FF47 1C       inc     dword ptr [edi+1C]
0040954E  |.   8B83 F4020000 mov     eax, [ebx+2F4]
00409554  |.   E8 53C10700   call    004856AC                 
00409559  |.   66:C747 10 08>mov     word ptr [edi+10], 8
0040955F  |.   837D FC 00    cmp     dword ptr [ebp-4], 0    ；这里判断密码是否为空
00409563  |.   74 05         je      short 0040956A
00409565  |.   8B4D FC       mov     ecx, [ebp-4]            ；当当当当当，我们输入密码出现
00409568  |.   EB 03         jmp     short 0040956D
0040956A  |>   8D4E 47       lea     ecx, [esi+47]
0040956D  |>   51            push    ecx                      ; /Arg2
0040956E  |.   53            push    ebx                      ; 将 8 L 和我们的密码传入
                                             ; 关键 call，我们跟入
0040956F  |.   E8 50FFFFFF   call    004094C4               ；很明显，这个是关键 call
00409574  |.   83C4 08       add     esp, 8
00409577  |.   3C 01         cmp     al, 1                     
00409579  |.   0F85 45010000 jnz     004096C4               ；这里跳转到下面的 badboy，上面正好有比较，我们先不让他跳，发现成功注册。
0040957F  |.   6A 40         push    40                   ；有同学就要说，直接爆破就好了嘛，其实这个是不行的，因为。。因为，他加壳了。
                                             ；我们现在改了，如果写入源文件的话，他再解压后就不能运行了。我们只能进行追码
                                             ；于是就产生了俺的第一个汇编的注册机，oh yeah
00409581  |.   8D86 A8000000 lea     eax, [esi+A8]
00409587  |.   50            push    eax
00409588  |.   8D56 48       lea     edx, [esi+48]
0040958B  |.   52            push    edx
0040958C  |.   8BC3          mov     eax, ebx
0040958E  |.   E8 2D280800   call    0048BDC0
00409593  |.   50            push    eax
00409594  |.   E8 DB560B00   call    004BEC74
00409599  |.   66:C747 10 2C>mov     word ptr [edi+10], 2C
0040959F  |.   33D2          xor     edx, edx
004095A1  |.   8955 F4       mov     [ebp-C], edx
004095A4  |.   8D55 F4       lea     edx, [ebp-C]
004095A7  |.   FF47 1C       inc     dword ptr [edi+1C]
004095AA  |.   A1 44A24C00   mov     eax, [4CA244]
004095AF  |.   8B00          mov     eax, [eax]
004095B1  |.   E8 BEE60600   call    00477C74
004095B6  |.   8D45 F4       lea     eax, [ebp-C]
004095B9  |.   8B00          mov     eax, [eax]
004095BB  |.   33D2          xor     edx, edx
004095BD  |.   8955 F0       mov     [ebp-10], edx
004095C0  |.   8D55 F0       lea     edx, [ebp-10]
004095C3  |.   FF47 1C       inc     dword ptr [edi+1C]
004095C6  |.   E8 E9840200   call    00431AB4
004095CB  |.   8D45 F0       lea     eax, [ebp-10]
004095CE  |.   50            push    eax
004095CF  |.   8D96 BB000000 lea     edx, [esi+BB]
004095D5  |.   8D45 EC       lea     eax, [ebp-14]
004095D8  |.   E8 6F440B00   call    004BDA4C
004095DD  |.   FF47 1C       inc     dword ptr [edi+1C]
004095E0  |.   33C0          xor     eax, eax
004095E2  |.   8945 F8       mov     [ebp-8], eax
004095E5  |.   8D55 EC       lea     edx, [ebp-14]
004095E8  |.   FF47 1C       inc     dword ptr [edi+1C]
004095EB  |.   8D4D F8       lea     ecx, [ebp-8]
004095EE  |.   58            pop     eax
004095EF  |.   E8 54460B00   call    004BDC48
004095F4  |.   FF4F 1C       dec     dword ptr [edi+1C]
004095F7  |.   8D45 EC       lea     eax, [ebp-14]
004095FA  |.   BA 02000000   mov     edx, 2
004095FF  |.   E8 EC450B00   call    004BDBF0
00409604  |.   FF4F 1C       dec     dword ptr [edi+1C]
00409607  |.   8D45 F0       lea     eax, [ebp-10]
0040960A  |.   BA 02000000   mov     edx, 2
0040960F  |.   E8 DC450B00   call    004BDBF0
00409614  |.   FF4F 1C       dec     dword ptr [edi+1C]
00409617  |.   8D45 F4       lea     eax, [ebp-C]
0040961A  |.   BA 02000000   mov     edx, 2
0040961F  |.   E8 CC450B00   call    004BDBF0
00409624  |.   66:C747 10 20>mov     word ptr [edi+10], 20
0040962A  |.   837D F8 00    cmp     dword ptr [ebp-8], 0
0040962E  |.   74 05         je      short 00409635
00409630  |.   8B4D F8       mov     ecx, [ebp-8]
00409633  |.   EB 06         jmp     short 0040963B
00409635  |>   8D8E D6000000 lea     ecx, [esi+D6]
0040963B  |>   51            push    ecx
0040963C  |.   837D FC 00    cmp     dword ptr [ebp-4], 0
00409640  |.   74 05         je      short 00409647
00409642  |.   8B45 FC       mov     eax, [ebp-4]
00409645  |.   EB 06         jmp     short 0040964D
00409647  |>   8D86 D5000000 lea     eax, [esi+D5]
0040964D  |>   50            push    eax
0040964E  |.   8D96 CE000000 lea     edx, [esi+CE]
00409654  |.   52            push    edx
00409655  |.   8D8E C7000000 lea     ecx, [esi+C7]
0040965B  |.   51            push    ecx
0040965C  |.   E8 2D500B00   call    004BE68E
00409661  |.   33D2          xor     edx, edx
00409663  |.   8B83 08030000 mov     eax, [ebx+308]
00409669  |.   8B08          mov     ecx, [eax]
0040966B  |.   FF51 64       call    [ecx+64]
0040966E  |.   66:C747 10 38>mov     word ptr [edi+10], 38
00409674  |.   8D96 D7000000 lea     edx, [esi+D7]
0040967A  |.   8D45 E8       lea     eax, [ebp-18]
0040967D  |.   E8 CA430B00   call    004BDA4C
00409682  |.   FF47 1C       inc     dword ptr [edi+1C]
00409685  |.   8B10          mov     edx, [eax]
```



### 00409687 - 0040971B 汇编代码段

```
00409687  |.  8B83 00030000 mov     eax, [ebx+300]
0040968D  |.  E8 4AC00700   call    004856DC
00409692  |.  FF4F 1C       dec     dword ptr [edi+1C]
00409695  |.  8D45 E8       lea     eax, [ebp-18]
00409698  |.  BA 02000000   mov     edx, 2
0040969D  |.  E8 4E450B00   call    004BDBF0
004096A2  |.  C783 4C020000>mov     dword ptr [ebx+24C], 1
004096AC  |.  FF4F 1C       dec     dword ptr [edi+1C]
004096AF  |.  8D45 F8       lea     eax, [ebp-8]
004096B2  |.  BA 02000000   mov     edx, 2
004096B7  |.  E8 34450B00   call    004BDBF0
004096BC  |.  66:C747 10 08>mov     word ptr [edi+10], 8
004096C2  |.  EB 38         jmp     short 004096FC            ；我们发现这个 jmp 可以跳过去
004096C4  |>  6A 10         push    10                        ；同时发现上面有一个跳转要调到这里，这条路一 badboy，于是向上看。
004096C6  |.  8D8E 10010000 lea     ecx, [esi+110]
004096CC  |.  51            push    ecx
004096CD  |.  8D86 DD000000 lea     eax, [esi+DD]
004096D3  |.  50            push    eax
004096D4  |.  8BC3          mov     eax, ebx
004096D6  |.  E8 E5260800   call    0048BDC0
004096DB  |.  50            push    eax
004096DC  |.  E8 93550B00   call    004BEC74                   ；跳出 MessageBox 的 call
004096E1  |.  FF4F 1C       dec     dword ptr [edi+1C]         ;这个是我们出来后的位置。
004096E4  |.  8D45 FC       lea     eax, [ebp-4]
004096E7  |.  BA 02000000   mov     edx, 2
004096EC  |.  E8 FF440B00   call    004BDBF0
004096F1  |.  8B0F          mov     ecx, [edi]
004096F3  |.  64:890D 00000>mov     fs:[0], ecx
004096FA  |.  EB 19         jmp     short 00409715
004096FC  |>  FF4F 1C       dec     dword ptr [edi+1C]
004096FF  |.  8D45 FC       lea     eax, [ebp-4]
00409702  |.  BA 02000000   mov     edx, 2
00409707  |.  E8 E4440B00   call    004BDBF0
0040970C  |.  8B0F          mov     ecx, [edi]
0040970E  |.  64:890D 00000>mov     fs:[0], ecx
00409715  |>  5F            pop     edi
00409716  |.  5E            pop     esi
00409717  |.  5B            pop     ebx
00409718  |.  8BE5          mov     esp, ebp
0040971A  |.  5D            pop     ebp
0040971B  \.  C3            retn
```

在上面函数的首部下硬件断点，重新运行。跟踪关键 `call`。

### 关键 Call 代码分析

关键 `call` 的代码如下（要考雅思啦，下面统一英文 comment）

```
004094C4  /$  55            push    ebp
004094C5  |.  8BEC          mov     ebp, esp
004094C7  |.  53            push    ebx
004094C8  |.  56            push    esi
004094C9  |.  57            push    edi
004094CA  |.  8B5D 0C       mov     ebx, [ebp+C]                 ；ebx==our key
004094CD  |.  85DB          test    ebx, ebx                     ；test if it is empty
004094CF  |.  74 0C         je      short 004094DD               ;exit if empty
004094D1  |.  53            push    ebx
004094D2  |.  E8 B1870A00   call    004B1C88                     ;  calculate the length of the key，
004094D7  |.  59            pop     ecx
004094D8  |.  83F8 10       cmp     eax, 10                      ;the length must be 10 in hex
004094DB  |.  74 04         je      short 004094E1               ;if the key is shorter or longer,exit
004094DD  |>  33C0          xor     eax, eax
004094DF  |.  EB 39         jmp     short 0040951A
004094E1  |>  0FBE73 06     movsx   esi, byte ptr [ebx+6]        ;fetch the sixth letter of our key
004094E5  |.  8BC6          mov     eax, esi
004094E7  |.  0FBE7B 0E     movsx   edi, byte ptr [ebx+E]        ;fetch the fourteenth letter of our key
004094EB  |.  03C7          add     eax, edi                     ;  add the sixth and fourteenth letter of the key
004094ED  |.  3D 9B000000   cmp     eax, 9B                      ;the sum must be 9b in hex
004094F2  |.  75 24         jnz     short 00409518
004094F4  |.  8BCE          mov     ecx, esi                     ;ecx==ascii of the sixth letter
004094F6  |.  2BCF          sub     ecx, edi                     ;sub the ascii of the fourteenth letter
004094F8  |.  8BC1          mov     eax, ecx                     ;put the answer into eax
004094FA  |.  99            cdq                                  ;edx is extended as the 31 number of eax
                                                                 ;if the highest bit of eax is 1,then
                                                                 ;edx==0FFFFh,else edx==0000h
004094FB  |.  33C2          xor     eax, edx                     ;let's assume the answer as x
                                                                 ;x xor edx
004094FD  |.  2BC2          sub     eax, edx                     ;(x xor edx)-edx
004094FF  |.  83C0 41       add     eax, 41                      ;(x xor edx)-edx +41
00409502  |.  0FBE53 09     movsx   edx, byte ptr [ebx+9]
00409506  |.  3BC2          cmp     eax, edx                     ;the ascii of the ningth letter must be {(x xor edx)-edx +41}
00409508  |.  75 0E         jnz     short 00409518               ;it's over
0040950A  |.  8B45 08       mov     eax, [ebp+8]                 ;  fetch its own key word
0040950D  |.  C680 34030000>mov     byte ptr [eax+334], 1
00409514  |.  B0 01         mov     al, 1
00409516  |.  EB 02         jmp     short 0040951A
00409518  |>  33C0          xor     eax, eax
0040951A  |>  5F            pop     edi
0040951B  |.  5E            pop     esi
0040951C  |.  5B            pop     ebx
0040951D  |.  5D            pop     ebp
0040951E  \.  C3            retn
```

根据我们的分析，创建 `keygen` 相当容易，所以让我们来做。

我会尽量把 `keygen` 做得简单。

我从 `pediy` 下载了一个 `asm` 模板，我会直接打包给你，你可以改变任何内容，就像我这样做一样，hehe，如果你不想这样做，你可以修改 `regkey.asm` 来制作一个新的 `keygen`，这里是我的 `regkey.asm` 源码。

### Keygen 源码 (regkey.asm)

```
include  string.inc
includelib string.lib

include  masm32.inc
includelib masm32.lib

include  debug.inc
includelib debug.lib

.data
.data?

szName  db 256 dup (?)
szSerial db 256 dup (?)

.code

;PrintHex edx
;DbgDump offset szName,16

GetRegCode proc hDlg
pushad

invoke  strempty,addr szSerial,sizeof szSerial
invoke GetDlgItemText,hDlg,IDC_NAME,addr szName,sizeof szName

.if eax
       mov ecx,6h
 mov esi,offset szName
 mov edi,offset szSerial
       rep movsb

 mov ebx,offset szSerial
 mov edx,'1'
 mov [ebx+6],edx

 mov ecx,'8'
 mov [ebx+7],ecx
 mov [ebx+8],ecx

 mov eax,'z'
 mov [ebx+9],eax

 mov eax,'s'
 mov [ebx+0ah],eax

 mov ecx,'s'
 mov [ebx+0bh],ecx

 mov ecx,'8'
 mov [ebx+0ch],ecx

 mov ecx,'3'
 mov [ebx+0dh],ecx

 mov ecx,'j'
 mov [ebx+0eh],ecx
 mov [ebx+0fh],ecx

 invoke SetDlgItemText,hDlg,IDC_REG,addr szSerial

.else
 invoke SetDlgItemText,hDlg,IDC_REG,CTXT("Just enter a five letter name！")

.endif
popad
ret

GetRegCode endp
```

我认为对于像你这样的天才来说，结合上述分析理解这个并不困难，所以，这里我就不注释了。

一个小技巧，如果你不小心注册了想要再玩一次，你可以直接删除 `setting.ini` 中的 `Serial`。

今天的部分就到这里！

祝好日子！

By `redblack`

`QQ:821038372`

### 附件

| 文件 | 说明 |
| :--- | :--- |
| `keygen-source.part1.rar` | |
| `keygen-source.part2.rar` | |
| `keygen.rar` | |
| `decrypt.pdf.rar` | |
