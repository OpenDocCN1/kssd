
# CacheX v4.00

> 原文：[`www.pediy.com/kssd/pediy04/kanxue048.htm`](https://www.pediy.com/kssd/pediy04/kanxue048.htm)

解除 `CacheX` for Internet Explorer V 4.00 的 `crc` 校验及破解，

发信人：玉川

整理时间：2001-09-18

软件介绍：

- `CacheX` for Internet Explorer
- 来自：Matthias Wolf, Albertinenstr
- 版本：`4.00`
- 最新更新：`2001-07-30`
- 文件大小：`296 KB`
- 软件授权：Shareware
- 作业平台：Win 95/98/NT/2000

可以在离线后轻松访问曾经浏览过的网页。包括全文检索，建立整个访问文档的结构。可以保存，隐藏和删除离线文档。当我们在线浏览网页时，浏览器自身已经下载了所需的页面和图片，在浏览器的缓存里面已经有很多我们需要的内容，如果能好好地利用它们，有时并不需要我们再次连线上网去查询和下载，那可是能节省不少银子的哟！`CacheX` 就是一个可以充分利用这些资源的离线浏览软件。

首先说明，我走的是一条最笨的破解之路，最便捷的还是破解它的注册码。这条弯弯拐拐的路可花了我不少时间。其中没有详细说明为何这样改，原因很简单，连我自己的笔记也不能看懂了，我也只能按笔记这样修改。不过，如果你有兴趣，按下面的顺序依次中断，就会发现东东。

还有一点想说的是，该软件的离线浏览功能制作没有它的防解密好。其实，我们用好 `IE` 的“历史”功能，同样很方便离线浏览。

首先，用 `topo` 打开 `cxie.exe`，我增加了 `300` 字节，但最好是增加 `400` 字节。得到：

```text
memory 0044a01d
file offset 00045e1d
```

以上得到的空间作为修改程序之用。如果你得到的地址与这不同，那下面修改部分的跳转地址就有所改变，请注意。

## 1. 调试器检测

```assembly
* Referenced by a CALL at Address:
|:0043DAE4 
|
:0043DAF0 56                   push esi
:0043DAF1 33C0                 xor eax, eax
:0043DAF3 8B742408             mov esi, dword ptr [esp+08]
:0043DAF7 57                   push edi
:0043DAF8 F7D6                 not esi
:0043DAFA 8B7C2410             mov edi, dword ptr [esp+10]
:0043DAFE F7D7                 not edi
:0043DB00 CC                   int 03
:0043DB01 5F                   pop edi
:0043DB02 40                   inc eax
:0043DB03 5E                   pop esi
:0043DB04 C3                   ret
```

## 2. crc 校验 （第一）

```assembly
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043D4A7(C)
|
:0043D4CA 8D85DCFAFFFF         lea eax, dword ptr [ebp+FFFFFADC]
:0043D4D0 50                   push eax
:0043D4D1 E854180000           call 0043ED2A
:0043D4D6 3B35A8294400         cmp esi, dword ptr [004429A8]
:0043D4DC 754A                 jne 0043D528                《----必须 nop 掉
:0043D4DE 8B45F0               mov eax, dword ptr [ebp-10]
:0043D4E1 F7D0                 not eax
:0043D4E3 3B0518E64300         cmp eax, dword ptr [0043E618]
:0043D4E9 753D                 jne 0043D528                《----必须 nop 掉
:0043D4EB 8B45E4               mov eax, dword ptr [ebp-1C]
:0043D4EE 8B4DE8               mov ecx, dword ptr [ebp-18]
:0043D4F1 2D008049B7           sub eax, B7498000
```

## 3. crc 校验 （第二）

```assembly
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043DF5E(C)
|
:0043DF7E 8D8B00100000         lea ecx, dword ptr [ebx+00001000]
:0043DF84 8D9348100000         lea edx, dword ptr [ebx+00001048]
:0043DF8A F7D0                 not eax                 《-------此处必须修改 eax 的值
:0043DF8C 8BF1                 mov esi, ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043DF9E(U)
|
:0043DF8E 3BCA                 cmp ecx, edx
:0043DF90 730E                 jnb 0043DFA0
:0043DF92 8B39                 mov edi, dword ptr [ecx]
:0043DF94 33F8                 xor edi, eax
```

在 `0043df8a` 处，必须修改 `eax` 的值，使 `eax=555200a3`，我这样修改：

```assembly
0043df7e e99ac00000    jmp 0044a01d
0043df83 90            nop
```

在 `0044a01d` 处：

```assembly
0044a01d 8d8b00100000    lea ecx,dword ptr [ebx+1000]
0044a023 b8a3005255      mov eax,555200a3
0044a028 e9573fffff      jmp 43df84
```

## 4. crc 校验 （第三）

```assembly
:0043EE28 6A00                   push 00000000
:0043EE2A 50                     push eax
:0043EE2B E836F4FFFF             call 0043E266
:0043EE30 FF75FC                 push [ebp-04]               《----修改此处
:0043EE33 8D459C                 lea eax, dword ptr [ebp-64]

* Possible Reference to Menu: MenuID_0001
                            |
* Possible Reference to String Resource ID=00001: "CacheX for Internet Explorer"
                            |
:0043EE36 6A01                   push 00000001

* Possible Reference to String Resource ID=00020: "Week of %s"
                            |
:0043EE38 6A14                   push 00000014
:0043EE3A 50                     push eax
:0043EE3B E891F0FFFF             call 0043DED1
:0043EE40 33C0                   xor eax, eax
:0043EE42 8D7D9C                 lea edi, dword ptr [ebp-64]
:0043EE45 AB                     stosd
:0043EE46 AB                     stosd
:0043EE47 AB                     stosd
:0043EE48 8B75F8                 mov esi, dword ptr [ebp-08]
:0043EE4B 53                     push ebx
:0043EE4C AB                     stosd
:0043EE4D AB                     stosd
:0043EE4E BFFAF64300             mov edi, 0043F6FA
:0043EE53 56                     push esi
:0043EE54 57                     push edi
:0043EE55 FF75FC                 push [ebp-04]
:0043EE58 E821F3FFFF             call 0043E17E
:0043EE5D FF75FC                 push [ebp-04]
:0043EE60 E84DF2FFFF             call 0043E0B2
:0043EE65 57                     push edi
:0043EE66 53                     push ebx
:0043EE67 56                     push esi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043EEDB(C)
|
:0043EE68 E8DAEFFFFF             call 0043DE47
:0043EE6D 85C0                   test eax, eax
:0043EE6F 7505                   jne 0043EE76
:0043EE71 E878E9FFFF             call 0043D7EE

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043EE6F(C)
|
:0043EE76 FF7510                 push [ebp+10]
:0043EE79 FF750C                 push [ebp+0C]
:0043EE7C FF7508                 push [ebp+08]
:0043EE7F FF75FC                 push [ebp-04]
:0043EE82 FFD6                   call esi              《---注意此 call,程序启动第一次中断在此，为利用注册名和注册码计算的 key 来解密已加密的程序代码，此 key 在[ebp+08]中的地址中。
:0043EE84 8BCB                   mov ecx, ebx
:0043EE86 894510                 mov dword ptr [ebp+10], eax
:0043EE89 8BD1                   mov edx, ecx
:0043EE8B 33C0                   xor eax, eax
:0043EE8D 8BFE                   mov edi, esi
:0043EE8F FF75FC                 push [ebp-04]
:0043EE92 C1E902                 shr ecx, 02
:0043EE95 F3                     repz
```

在 `43ee30` 处，改为：

```assembly
0043ee30 e9fbb10000      jmp 0044a030
0043ee35 90              nop
```

在 `0044a030` (offset `45e30`) 处，改为：

```assembly
0044a030 ff75fc          push [ebp-04]
0044a033 8d459c          lea eax,dword ptr [ebp-64]
0044a036 c700bc6c4684    mov [eax],84466cbc
0044a03c c7400481ff001e  mov [eax+04],1e00ff81
0044a043 c740085c3ae70e  mov [eax+08],0ee73a5c
0044a04a c7400c74c49c35  mov [eax+0c],359cc474
0044a051 c74010f7fc284b  mov [eax+10],4b28fcf7
0044a058 e9d84dffff      jmp 0043ee36
```

## 5. crc 校验 （第四）

```assembly
:0043D0CA 50                   push eax
:0043D0CB E896110000           call 0043E266
:0043D0D0 53                   push ebx

* Possible Reference to Menu: MenuID_0001
                            |
* Possible Reference to String Resource ID=00001: "CacheX for Internet Explorer"
                            |
:0043D0D1 6A01                 push 00000001

* Possible Reference to String Resource ID=00020: "Week of %s"
                            |
:0043D0D3 6A14                 push 00000014            《----修改此处
:0043D0D5 8D4590               lea eax, dword ptr [ebp-70]
:0043D0D8 50                   push eax
:0043D0D9 E8F30D0000           call 0043DED1
:0043D0DE 33C0                 xor eax, eax
:0043D0E0 8D7DEC               lea edi, dword ptr [ebp-14]
:0043D0E3 AB                   stosd
```



### 1. 初始修改

:0043D0E4 `AB`    `stosd`   

:0043D0E5 `AB`    `stosd`   

:0043D0E6 `AB`    `stosd`   

:0043D0E7 `33C0`  `xor eax, eax`   

在 `43d0d3` 处，如下修改：   

:0043d0d3 `e988cf0000`  `jmp 0044a060`   

在 `0044a060` (offset `45e60`)处打入如下数据:   

```
6a148d4590
c7006bba11d0
c74004e8c340bb
c74008afaf5906
c7400c09e66d52
c7401040e6efbb
e94c30ffff
```

### 6. crc 校验 (第五)

:0040BB86 `8D4590`  `lea eax, dword ptr [ebp-70]`   

:0040BB89 `53`      `push ebx`   

:0040BB8A `50`      `push eax`   

:0040BB8B `E8D6260300`  `call 0043E266`   

:0040BB90 `FF75FC`  `push [ebp-04]`   《----修改此处   

:0040BB93 `8D4590`  `lea eax, dword ptr [ebp-70]`   

:0040BB96 `57`      `push edi`   

* Possible Reference to String Resource `ID=00020`: `"Week of %s"`   

|   

:0040BB97 `6A14`    `push 00000014`   

:0040BB99 `50`      `push eax`   

:0040BB9A `E832230300`  `call 0043DED1`   

:0040BB9F `8D45EC`  `lea eax, dword ptr [ebp-14]`   

* Possible Reference to Dialog: `DialogID_014D`, `CONTROL_ID:0010`, `"Image and media files:"`   

|   

* Possible Reference to String Resource `ID=00016`: `" (not in Master Category List)"`   

|   

:0040BBA2 `6A10`    `push 00000010`   

:0040BBA4 `50`      `push eax`   

:0040BBA5 `6898864400`  `push 00448698`   

在 `0040bb90` 处，如下修改:   

:0040bb90 `e9f7e40300`  `jmp 0044a08c`   
:0040bb95 `90`  `nop`   

在 `0044a08c` 处(offset `45e8c`)打入如下数据:   

```
ff75fc8d4590
c700f4622151
c740046717c3d7
c7400831aa0084
c7400cbefe6e71
c740109e675f97
e9dd1afcff
```

### 7. crc 校验 （第六）

:0040BC2D `53`      `push ebx`   

:0040BC2E `50`      `push eax`   

:0040BC2F `E832260300`  `call 0043E266`   

:0040BC34 `8B7DFC`  `mov edi, dword ptr [ebp-04]`  《---修改此处   

:0040BC37 `8D4590`  `lea eax, dword ptr [ebp-70]`   

:0040BC3A `57`      `push edi`   

* Possible Reference to Menu: `MenuID_0001`   

|   

* Possible Reference to String Resource `ID=00001`: `"CacheX for Internet Explorer"`   

|   

:0040BC3B `6A01`    `push 00000001`   

* Possible Reference to String Resource `ID=00020`: `"Week of %s"`   

|   

:0040BC3D `6A14`    `push 00000014`   

:0040BC3F `50`      `push eax`   

:0040BC40 `E88C220300`  `call 0043DED1`   

* Possible Reference to Dialog: `DialogID_014D`, `CONTROL_ID:0010`, `"Image and media files:"`   

|   

* Possible Reference to String Resource `ID=00016`: `" (not in Master Category List)"`   

|   

:0040BC45 `6A10`    `push 00000010`   

:0040BC47 `8D45EC`  `lea eax, dword ptr [ebp-14]`   

:0040BC4A `53`      `push ebx`   

在 `0040bc34` 处，如下修改:   

:0040bc34 `e980e40300`  `jmp 0044a0b9`   
:0040bc39 `90`  `nop`   

在 `0044a0b9` 处(offset `45eb9`)打入如下数据:   

```
8b7dfc8d4590
c700670b6c92
c740041abcfe56
c7400894914c33
c7400cc7ce935c
c7401099aca6b8
e9541bfcff
```

### 8. crc 校验 (第七)

:00418CCA `50`      `push eax`   

:00418CCB `E896550200`  `call 0043E266`   

:00418CD0 `53`      `push ebx`   

* Possible Reference to Menu: `MenuID_0001`   

|   

* Possible Reference to String Resource `ID=00001`: `"CacheX for Internet Explorer"`   

|   

:00418CD1 `6A01`    `push 00000001`  《----修改此处   

:00418CD3 `8D4590`  `lea eax, dword ptr [ebp-70]`   

* Possible Reference to String Resource `ID=00020`: `"Week of %s"`   

|   

:00418CD6 `6A14`    `push 00000014`   

:00418CD8 `50`      `push eax`   

:00418CD9 `E8F3510200`  `call 0043DED1`   

:00418CDE `8945FC`  `mov dword ptr [ebp-04], eax`   

* Possible Reference to String Resource `ID=00020`: `"Week of %s"`   

|   

:00418CE1 `6A14`    `push 00000014`   

在 `00418cd1` 处如下修改:   

:00418cd1 `e910140300`  `jmp 0044a0e6`   

在 `0044a0e6` 处(offset `45ee6`)打入如下数据:   

```
6a018d4590
c700fa4614e4
c74004bd5ca7ea
c74008728b6d4f
c7400cf2c711a8
c74010907874c3
e9c4ebfcff
```

### 9. crc 校验 (第八)

:00421891 `72DD`    `jb 00421870`   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0042186E(C)   

|   

:00421893 `8B45F8`  `mov eax, dword ptr [ebp-08]`  《----修改此处   

:00421896 `8BD7`    `mov edx, edi`   

:00421898 `F7D1`    `not ecx`   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:004218AC(C)   

|   

:0042189A `8B3A`    `mov edi, dword ptr [edx]`   

:0042189C `83C204`  `add edx, 00000004`   

:0042189F `33F9`    `xor edi, ecx`   

:004218A1 `8938`    `mov dword ptr [eax], edi`   

:004218A3 `83C004`  `add eax, 00000004`   

:004218A6 `81FA7C244400`  `cmp edx, 0044247C`   

:004218AC `72EC`    `jb 0042189A`   

:004218AE `8B86E8060000`  `mov eax, dword ptr [esi+000006E8]`   

在 `00421893` 处如下修改:   

:0042893 `e97a880200`  `jmp 0044a112`   

在 `0044a112` 处(offset `45f12`)打入如下数据:   

```
8b45f88bd7b9a3005255e97777fdff
```

### 10. crc 校验 (第九)

:004046F8 `53`      `push ebx`   

:004046F9 `68B8074400`  `push 004407B8`  《----修改此处   

:004046FE `FF75F8`  `push [ebp-08]`   

:00404701 `E88A410300`  `call 00438890`   

:00404706 `8B8680000000`  `mov eax, dword ptr [esi+00000080]`   

:0040470C `83C40C`  `add esp, 0000000C`   

:0040470F `8BD0`    `mov edx, eax`   

:00404711 `8D8890000000`  `lea ecx, dword ptr [eax+00000090]`   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:00404728(U)   

|   

:00404717 `3BC1`    `cmp eax, ecx`   

:00404719 `730F`    `jnb 0040472A`   

:0040471B `8B18`    `mov ebx, dword ptr [eax]`   

:0040471D `335DFC`  `xor ebx, dword ptr [ebp-04]`   

:00404720 `891A`    `mov dword ptr [edx], ebx`   

:00404722 `83C204`  `add edx, 00000004`   

:00404725 `83C004`  `add eax, 00000004`   

:00404728 `EBED`    `jmp 00404717`   

在 `004046f9` 处如下修改:   

:004046f9 `e9235a0400`  `jmp 0044a121`   

在 `0044a121` 处(offset `45f21`)打入如下数据:   

```
68b8074400c745fca2461300e9cca5fbff
```

### 11. 关于 delete 的破解

:0041C582 `33C8`    `xor ecx, eax`   

:0041C584 `FF45FC`  `inc [ebp-04]`   

:0041C587 `817DFCF4EE4300`  `cmp dword ptr [ebp-04], 0043EEF4`   

:0041C58E `72D9`    `jb 0041C569`   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0041C567(C)   

|   

:0041C590 `57`      `push edi`      《----修改此处   

:0041C591 `53`      `push ebx`   

:0041C592 `FF75F8`  `push [ebp-08]`   

:0041C595 `F7D1`    `not ecx`   

:0041C597 `894DFC`  `mov dword ptr [ebp-04], ecx`   

:0041C59A `E8F1C20100`  `call 00438890`   

:0041C59F `8B4658`  `mov eax, dword ptr [esi+58]`   

:0041C5A2 `83C40C`  `add esp, 0000000C`   

:0041C5A5 `8BD0`    `mov edx, eax`   

:0041C5A7 `8D8890000000`  `lea ecx, dword ptr [eax+00000090]`   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0041C5BE(U)   

|   

:0041C5AD `3BC1`    `cmp eax, ecx`   



```
:0041C5AF 730F                  jnb 0041C5C0
:0041C5B1 8B38                  mov edi, dword ptr [eax]
:0041C5B3 337DFC                xor edi, dword ptr [ebp-04]
:0041C5B6 893A                  mov dword ptr [edx], edi
:0041C5B8 83C204                add edx, 00000004
:0041C5BB 83C004                add eax, 00000004
:0041C5BE EBED                  jmp 0041C5AD
```

在`41c590`处，作如下修改:

```
0041c590 e99ddb0200 jmp 44a132
```

然后在`44a132`处(offset`45f32`)打入：

```
5753ff75f8b9a3005255e95424fdff
```

这样修改后，`delete`不会出错了，但是，程序会退出。可以肯定，还有一个更狡猾的`crc`校验。我跟踪了一阵，如下代码可疑:

```
:004079E6 FF742410              push [esp+10]
:004079EA B9607C4400            mov ecx, 00447C60
:004079EF 55                    push ebp
:004079F0 E8F59CFFFF            call 004016EA
:004079F5 8B742418              mov esi, dword ptr [esp+18]
:004079F9 50                    push eax
:004079FA 8BCE                  mov ecx, esi
:004079FC E8694C0100            call 0041C66A          《---此处可疑,里面的代码很长。
:00407A01 6AFF                  push FFFFFFFF
:00407A03 8BCF                  mov ecx, edi
:00407A05 E865000200            call 00427A6F          《---此处可疑,里面的代码很长。
:00407A0A 8B06                  mov eax, dword ptr [esi]
:00407A0C 3BDD                  cmp ebx, ebp
:00407A0E 55                    push ebp
:00407A0F 7507                  jne 00407A18
:00407A11 8BCE                  mov ecx, esi
:00407A13 FF5074                call [eax+74]
:00407A16 EB06                  jmp 00407A1E
```

不知正确与否，望有兴趣的朋友仔细研究一下，指教小弟几招.

小弟无能，仍然没能完全搞定`cachex`,望各位赐教.

12、通过以上`crc`校验修改后，便可对程序进行任意修改了.

除去时间限制:

```
:0043D70C 0FB705A8864400        movzx eax, word ptr [004486A8]
:0043D713 0FB70DB8864400        movzx ecx, word ptr [004486B8]
:0043D71A 2BC8                  sub ecx, eax
:0043D71C 780B                  js 0043D729
:0043D71E 0FBE0590864400        movsx eax, byte ptr [00448690]
:0043D725 2BC1                  sub eax, ecx          《----eax 中为 2e,ecx 为已使用天数.
:0043D727 7902                  jns 0043D72B          《---改为 jmp

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043D71C(C)
|
:0043D729 33C0                  xor eax, eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043D727(C)
|
:0043D72B C3                    ret
```

在`0043d725`处,随你改，我将`2bc1`改为`b008`,这样每每都有8天的使用时间,永不过期.`0043d727`必改.

通过上面的时间限制修改，仍有未注册的时间提示窗口，如你不满意的话，想去掉该窗口，可作如下修改:

```
:0043D13C 8BC3                  mov eax, ebx
:0043D13E 2BC7                  sub eax, edi
:0043D140 50                   push eax
:0043D141 FF750C                push [ebp+0C]         《----修改此处
:0043D144 FF7508                push [ebp+08]
:0043D147 FFD3                 call ebx
:0043D149 8BCE                  mov ecx, esi
:0043D14B 89450C                mov dword ptr [ebp+0C], eax
:0043D14E 8BD1                  mov edx, ecx
:0043D150 33C0                  xor eax, eax
:0043D152 8BFB                  mov edi, ebx
:0043D154 53                    push ebx
:0043D155 C1E902                shr ecx, 02
:0043D158 F3                    repz
:0043D159 AB                    stosd
:0043D15A 8BCA                  mov ecx, edx
:0043D15C 83E103                and ecx, 00000003
:0043D15F F3                    repz
:0043D160 AA                    stosb
:0043D161 E8FFAAFEFF            call 00427C65
:0043D166 8B450C                mov eax, dword ptr [ebp+0C]
:0043D169 59                    pop ecx
:0043D16A 5F                    pop edi
:0043D16B 5E                    pop esi
:0043D16C 5B                    pop ebx
:0043D16D C9                    leave
:0043D16E C20800                ret 0008
```

在`0043d141`处，将`ff750c`改为`6a0390`即可.但这样一来，`"about"`的窗口将不再出现.

但在标题栏仍有`Unregistered Version`字样，如何将`(Unregistered Version)`去掉呢？好办，用`winhex`打开`CXie.exe`,

在offset`5bb14`处，看到了吧,将`Unregistered Version`改为你喜欢的东东。我改为了`(version 4.00)`

通过这样修改，除了`delete`外，其它功能应该没有问题.唯一遗憾的是`delete`还未解决,不过我正在努力。我相信，没有破不了的东东。(注意那个`about`窗口)

感谢你花了这么长时间看到这里，望及时指教.

玉川

2001.09.18 `4dd466d0e1e0cc123afbe504da2462ca`
