
# 黄河 Flash 播放器 v7.38

> 原文：[`www.pediy.com/kssd/pediy05/pediy50561.htm`](https://www.pediy.com/kssd/pediy05/pediy50561.htm)

- 下载页面： [`www.skycn.com/soft/9167.html`](http://www.skycn.com/soft/9167.html)
- 软件大小： 2529 KB
- 软件语言： 简体中文
- 软件类别： 国产软件 / 免费版 / 图像浏览
- 应用平台： `Win9x/NT/2000/XP`
- 加入时间： 2003-03-16 15:37:12
- 下载次数： 46536
- 推荐等级： ***
- 开发商： [`huanghetv.diy.163.com/`](http://huanghetv.diy.163.com/)

【软件简介】：黄河 Flash 播放器，可以播放各类 Flash 文件，使用超酷界面，可保存列表文件、打开开列表文件、清空列表，具有以下功能：1、启动程序，按打开按钮，选择要播放的文件；2、在不同的地方按鼠标右键，有不同的功能；3、可以保存播放列表，也可以打开播放列表；4、文件夹中双击 `swf` 或 `hhf`(黄河 Flash 播放器列表文件)文件，可以自动播放放；5、可以无级缩放播放大小；6、显示播放进度，可以拖放！.......可以让你一边工作，一边欣赏动画，该软件目前没有任何注册限制可免费使用。

【软件限制】：注册后将无限免费发送本软件的最新版！

【作者声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

【破解工具】：`TRW2000` 娃娃修改版、`Ollydbg1.09`、`PEiD`、`AspackDie`、`W32Dasm 10` 修改版

---

## 【过 程】：

呵呵，看到 透明的 朋友有篇教程，正好从《软件王》光盘里发现这个程序，索性我试试其算法吧。

需要说明的是：天空上的是新版 V7.60 正式版，而我手里分析的是 V7.38 版，可能有些地方是不同的。

`hhflash.exe` 是 `ASPack 2.12` 壳，用 `AspackDie` 脱之。`629K->1.3M`。`Delphi` 编写。

- 识别号：`ID555490825`
- 试炼码：`13572468`

反汇编，根据出错提示很容易就找到下面的核心了。

```asm
:004E8259 E816F4F5FF           call 00447674
:004E825E 8B45F8               mov eax, dword ptr [ebp-08]
:004E8261 8D55FC               lea edx, dword ptr [ebp-04]
:004E8264 E86F0CF2FF           call 00408ED8
:004E8269 8B45FC               mov eax, dword ptr [ebp-04]
:004E826C 50                   push eax
:004E826D 8D55F0               lea edx, dword ptr [ebp-10]
:004E8270 8B83FC020000         mov eax, dword ptr [ebx+000002FC]
:004E8276 E8F9F3F5FF           call 00447674
:004E827B 8B45F0               mov eax, dword ptr [ebp-10]
                                 ==== >EAX=ID555490825       识别号
:004E827E 8D55F4               lea edx, dword ptr [ebp-0C]
:004E8281 E876E2FFFF           call 004E64FC
                                 ==== >算法 CALL
:004E8286 8B55F4               mov edx, dword ptr [ebp-0C]
                                 ==== >EDX=XL1rpw2wT82ALRg3SmPdlTgGsdEU8P
:004E8289 58                   pop eax
                                 ==== >EAX=13572468          试炼码
:004E828A E8B1CAF1FF           call 00404D40
                                 ==== >比较 CALL
:004E828F 7534                 jne 004E82C5
                                 ==== >跳则 OVER！

* Possible StringData Ref from Code Obj ->"注册成功！"
                                 ==== >呵呵，胜利女神！
                                  |
:004E8291 B824834E00           mov eax, 004E8324
:004E8296 E8F581F5FF           call 00440490
:004E829B 8D55E8               lea edx, dword ptr [ebp-18]
:004E829E 8B8304030000         mov eax, dword ptr [ebx+00000304]
:004E82A4 E8CBF3F5FF           call 00447674
:004E82A9 8B45E8               mov eax, dword ptr [ebp-18]
:004E82AC 8D55EC               lea edx, dword ptr [ebp-14]
:004E82AF E8240CF2FF           call 00408ED8
:004E82B4 8B45EC               mov eax, dword ptr [ebp-14]
:004E82B7 E86CFEFFFF           call 004E8128
:004E82BC 8BC3                 mov eax, ebx
:004E82BE E861CBF7FF           call 00464E24
:004E82C3 EB0A                 jmp 004E82CF

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E828F(C)
|

* Possible StringData Ref from Code Obj ->"注册号不对!"
                                 ==== >BAD BOY！
                                  |
:004E82C5 B838834E00           mov eax, 004E8338
:004E82CA E8C181F5FF           call 00440490

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E82C3(U)
|
:004E82CF 33C0                 xor eax, eax
:004E82D1 5A                   pop edx
:004E82D2 59                   pop ecx
:004E82D3 59                   pop ecx
:004E82D4 648910               mov dword ptr fs:[eax], edx
:004E82D7 6814834E00           push 004E8314

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E8312(U)
|
:004E82DC 8D45E8               lea eax, dword ptr [ebp-18]
:004E82DF E860C6F1FF           call 00404944
:004E82E4 8D45EC               lea eax, dword ptr [ebp-14]
:004E82E7 E858C6F1FF           call 00404944
:004E82EC 8D45F0               lea eax, dword ptr [ebp-10]
:004E82EF E850C6F1FF           call 00404944
:004E82F4 8D45F4               lea eax, dword ptr [ebp-0C]
:004E82F7 E848C6F1FF           call 00404944
:004E82FC 8D45F8               lea eax, dword ptr [ebp-08]
:004E82FF E840C6F1FF           call 00404944
:004E8304 8D45FC               lea eax, dword ptr [ebp-04]
:004E8307 E838C6F1FF           call 00404944
:004E830C C3                   ret
```

进入算法 CALL：`4E8281 call 004E64FC`

```asm
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E64FA(C)
|
:004E64FC 55                   push ebp
:004E64FD 8BEC                 mov ebp, esp
:004E64FF 83C4F0               add esp, FFFFFFF0
:004E6502 53                   push ebx
:004E6503 56                   push esi
:004E6504 57                   push edi
:004E6505 33C9                 xor ecx, ecx
:004E6507 894DF0               mov dword ptr [ebp-10], ecx
:004E650A 8BFA                 mov edi, edx
:004E650C 8945FC               mov dword ptr [ebp-04], eax
:004E650F 8B45FC               mov eax, dword ptr [ebp-04]
                                 ==== >EAX=ID555490825
:004E6512 E8CDE8F1FF           call 00404DE4
:004E6517 33C0                 xor eax, eax
:004E6519 55                   push ebp
:004E651A 68FF654E00           push 004E65FF
:004E651F 64FF30               push dword ptr fs:[eax]
:004E6522 648920               mov dword ptr fs:[eax], esp
:004E6525 8BC7                 mov eax, edi
:004E6527 E818E4F1FF           call 00404944
:004E652C 8D45FC               lea eax, dword ptr [ebp-04]
:004E652F BA18664E00           mov edx, 004E6618
:004E6534 E8CBE6F1FF           call 00404C04
:004E6539 8B45FC               mov eax, dword ptr [ebp-04]
:004E653C 0FB630               movzx esi, byte ptr [eax]
                                 1、  ====>ESI=49
:004E653F 666BC664             imul ax, si, 0064
                                 1、  ====>AX=49 * 64=1C84
:004E6543 668945F8             mov word ptr [ebp-08], ax
:004E6547 8BC6                 mov eax, esi
:004E6549 C1E009               shl eax, 09
                                 1、  ====>EAX=49  SHL  09=9200 
:004E654C 668945F6             mov word ptr [ebp-0A], ax
:004E6550 B301                 mov bl, 01

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E65DB(C)
|
:004E6552 8B45FC               mov eax, dword ptr [ebp-04]
:004E6555 E8A2E6F1FF           call 00404BFC
:004E655A 50                   push eax
:004E655B 33C0                 xor eax, eax
:004E655D 8AC3                 mov al, bl
:004E655F 5A                   pop edx
:004E6560 8BCA                 mov ecx, edx
:004E6562 99                   cdq
:004E6563 F7F9                 idiv ecx
:004E6565 8B45FC               mov eax, dword ptr [ebp-04]
                                 ==== >EAX=ID555490825

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆
EAX 内存中的值：
00CB15B0  49 44 35 35 35 34 39 30 38 32 35 01      ID555490825
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

:004E6568 8A0410               mov al, byte ptr [eax+edx]
                                 ==== >循环取 00CB15B0 内存处字符的 HEX 值
                                 1、  ====>AL=44
```



2、`====>AL=35`

3、`====>AL=35`

…… ……省 略…… ……

`:004E656B 0FB7D6` `movzx edx, si`

1、`====>EDX=49`

2、`====>EDX=8104`

3、`====>EDX=FF40`

…… ……省 略…… ……

`:004E656E C1EA08` `shr edx, 08`

1、`====>EDX=49` `SHR 08=00`

2、`====>EDX=8104` `SHR 08=81`

3、`====>EDX=FF40` `SHR 08=FF`

…… ……省 略…… ……

`:004E6571 32C2` `xor al, dl`

1、`====>AL=44` `XOR 00=44`

2、`====>AL=35` `XOR 81=B4`

3、`====>AL=35` `XOR FF=CA`

…… ……省 略…… ……

`:004E6573 33D2` `xor edx, edx`

`:004E6575 8AD3` `mov dl, bl`

`* Possible StringData Ref from Code Obj ->"Qwert yuiop asdfg hjkl zxcvbnm"`

|

`:004E6577 B924664E00` `mov ecx, 004E6624`

`====>ECX=Qwert yuiop asdfg hjkl zxcvbnm`

`:004E657C 324411FF` `xor al, byte ptr [ecx+edx-01]`

`AL` 与 `Qwert yuiop asdfg hjkl zxcvbnm` 中的字符值依次异或

1、`====>AL=44` `XOR 51=15`

2、`====>AL=B4` `XOR 77=C3`

3、`====>AL=CA` `XOR 65=AF`

…… ……省 略…… ……

`:004E6580 50` `push eax`

`:004E6581 8B45FC` `mov eax, dword ptr [ebp-04]`

`:004E6584 E873E6F1FF` `call 00404BFC`

`:004E6589 5A` `pop edx`

`:004E658A 02D0` `add dl, al`

1、`====>DL=15` `+ 0C=21`

2、`====>DL=C3` `+ 0C=CF`

3、`====>DL=AF` `+ 0C=BB`

…… ……省 略…… ……

`:004E658C 8855FB` `mov byte ptr [ebp-05], dl`

`:004E658F 33C0` `xor eax, eax`

`:004E6591 8A45FB` `mov al, byte ptr [ebp-05]`

`:004E6594 B93E000000` `mov ecx, 0000003E`

`:004E6599 99` `cdq`

`:004E659A F7F9` `idiv ecx`

1、`====>EDX=21` `% 3E=21`

2、`====>EDX=CF` `% 3E=15`

3、`====>EDX=BB` `% 3E=1`

…… ……省 略…… ……

`:004E659C 42` `inc edx`

`====>EDX 增 1`

`:004E659D 8855FB` `mov byte ptr [ebp-05], dl`

`:004E65A0 8D45F0` `lea eax, dword ptr [ebp-10]`

`:004E65A3 33D2` `xor edx, edx`

`:004E65A5 8A55FB` `mov dl, byte ptr [ebp-05]`

`* Possible StringData Ref from Code Obj ->"0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghij"`

`->"klmnopqrstuvwxyz"`

|

`:004E65A8 B94C664E00` `mov ecx, 004E664C`

`====>ECX=0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz`

`:004E65AD 8A5411FF` `mov dl, byte ptr [ecx+edx-01]`

`根据 EDX 的值，从 0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz 表中不同位置取字符`

1、`====>DL=58` 即：字符 `X`。取第 33 位

2、`====>DL=4C` 即：字符 `L`。取第 22 位

3、`====>DL=31` 即：字符 `1`。取第 2 位

…… ……省 略…… ……

`循环 30 次后所得字符： XL1rpw2wT82ALRg3SmPdlTgGsdEU8P 就是注册码了！`

`:004E65B1 E86EE5F1FF` `call 00404B24`

`:004E65B6 8B55F0` `mov edx, dword ptr [ebp-10]`

`:004E65B9 8BC7` `mov eax, edi`

`:004E65BB E844E6F1FF` `call 00404C04`

`:004E65C0 33C0` `xor eax, eax`

`:004E65C2 8AC3` `mov al, bl`

`:004E65C4 8B17` `mov edx, dword ptr [edi]`

`:004E65C6 0FB64402FF` `movzx eax, byte ptr [edx+eax-01]`

`:004E65CB 6603F0` `add si, ax`

1、`====>SI=49` `+ 58=A1`

2、`====>SI=8104` `+ 4C=8150`

3、`====>SI=FF40` `+ 31=FF71`

…… ……省 略…… ……

`:004E65CE 660FAF75F8` `imul si, word ptr [ebp-08]`

1、`====>SI=A1` `* 1C84=EF04`

2、`====>SI=8150` `* 1C84=6D40`

3、`====>SI=FF71` `* 1C84=1244`

…… ……省 略…… ……

`:004E65D3 660375F6` `add si, word ptr [ebp-0A]`

1、`====>SI=EF04` `+ 9200=8104`

2、`====>SI=6D40` `+ 9200=FF40`

3、`====>SI=1244` `+ 9200=A444`

…… ……省 略…… ……

`:004E65D7 43` `inc ebx`

`:004E65D8 80FB1F` `cmp bl, 1F`

`:004E65DB 0F8571FFFFFF` `jne 004E6552`

`====>循环 30 次！得到 30 个字符的注册码！`

`:004E65E1 33C0` `xor eax, eax`

`:004E65E3 5A` `pop edx`

`:004E65E4 59` `pop ecx`

`:004E65E5 59` `pop ecx`

`:004E65E6 648910` `mov dword ptr fs:[eax], edx`

`:004E65E9 6806664E00` `push 004E6606`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`|:004E6604(U)`

`|`

`:004E65EE 8D45F0` `lea eax, dword ptr [ebp-10]`

`:004E65F1 E84EE3F1FF` `call 00404944`

`:004E65F6 8D45FC` `lea eax, dword ptr [ebp-04]`

`:004E65F9 E846E3F1FF` `call 00404944`

`:004E65FE C3` `ret`

—————————————————————————————————

### 【KeyMake 之内存注册机】

- 中断地址：`4E8289`
- 中断次数：`1`
- 第一字节：`58`
- 指令长度：`1`
- 内存方式：`EDX`

—————————————————————————————————

### 【注册信息保存】

同文件夹下的 `hhflash.ini` 中：

```ini
[Mainform]
XpMenu=True
top=85
left=595
RegCode=XL1rpw2wT82ALRg3SmPdlTgGsdEU8P
```

—————————————————————————————————

### 【整理】

- 识别号：`ID555490825`
- 注册码：`XL1rpw2wT82ALRg3SmPdlTgGsdEU8P`

—————————————————————————————————

Cracked By 巢水工作坊——fly【OCN】

2003-4-10 13:31 `4dd466d0e1e0cc123afbe504da2462ca`
