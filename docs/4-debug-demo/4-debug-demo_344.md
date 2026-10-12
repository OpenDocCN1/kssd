
# rhino V2.03 注册分析 [KeyFile protect]

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-236.htm`](https://www.pediy.com/kssd/pediy07/pediy7-236.htm)

## 基本信息

【破解作者】 `winndy` `[FCG]` `[PYG]`  
【作者邮箱】 `CNwinndy@hotmail.com`  
【使用工具】 `PEID` `v0.93`、`OllyDbg` `v1.10`、`fly` 修改版、`upx` `1.24w`  
【破解平台】 `Winxp` `SP2`  
【软件名称】 `rhino` `V2.03`  
【官方网址】 `http://bigtick.pastnotecut.org`  
【编写语言】 `Microsoft` `Visual` `C++`

## 破解说明

keyfile 保护，失误之处还请指教！

在浏览 `http://www.codecomments.com/archive258-2004-4-152138.html` 的时候，看到：

> BigTick
> 2004-03-19, 8:26 pm
> I've written a short essay that may interest you. It's mostly C++, but it shows that you can get a decent amount of protection with only a little bit of asm. Of course if you code primarily in asm, you can apply the same technique.
> Anyway... read the stuff here: `http://bigtick.pastnotecut.org/tutorials/cp.html`
> Comments are welcome.
> 'Tick
> PS. The text describes the principles only, but I can provide a fully working example if you're interested.

这样一段话，不禁给人跃跃欲试的感觉。看了他的那篇文章，还不错，于是 down 下 `rhino` 看看。

【破解声明】 For study ,For Fun

## 破解过程

`PEID` 查壳，`UPX` `0.89.6` - `1.02` / `1.05` - `1.24` -> `Markus` & `Laszlo`

用 `upx` `1.24w` 脱壳：
`upx -d -o unpacked.exe rhino2.exe`

用 `OD` 载入 `unpacked.exe`,搜索字符串：

* 文本字符串参考位于 `unpacked:.text`，项目 `2241`
  * 地址=`00451972`
  * 反汇编=`push unpacked.004B7F74`
  * 文本字符=`ASCII "Thank you for using Rhino !"`
* 文本字符串参考位于 `unpacked:.text`，项目 `2242`
  * 地址=`004519E6`
  * 反汇编=`push unpacked.004B7F38`
  * 文本字符=`ASCII "Please support shareware development by registering Rhino."`

下断，并双击

```
代码:

* * *

  00451935     > \8B55 F0           mov edx,dword ptr ss:[ebp-10]
  00451938     .  8B8A 48080000    mov ecx,dword ptr ds:[edx+848]
  0045193E     .  E8 CDB0FDFF      call unpacked.0042CA10            ==>Attention
  00451943       0FB6C0            movzx eax,al
  00451946       85C0             test eax,eax                      ===>注意
  00451948     .  74 74            je short unpacked.004519BE
  0045194A     .  8D8D 97FCFFFF    lea ecx,dword ptr ss:[ebp-369]
  00451950     .  E8 1B280000     call unpacked.00454170
  00451955     .  8985 E8FBFFFF    mov dword ptr ss:[ebp-418],eax
  0045195B     .  8B8D E8FBFFFF    mov ecx,dword ptr ss:[ebp-418]
  00451961     .  898D E4FBFFFF    mov dword ptr ss:[ebp-41C],ecx
  00451967     .  C645 FC 22       mov byte ptr ss:[ebp-4],22
  0045196B     .  8B95 E4FBFFFF    mov edx,dword ptr ss:[ebp-41C]
  00451971     .  52               push edx                            ; /Arg2
  00451972     .  68 747F4B00      push unpacked.004B7F74              ; |Arg1 = 004B7F74 ASCII "Thank you for using Rhino !"
  00451977     .  8D8D 98FCFFFF    lea ecx,dword ptr ss:[ebp-368]      ; |
  0045197D     .  E8 5E9FFDFF      call unpacked.0042B8E0             ; \unpacked.0042B8E0
  00451982     .  C645 FC 23       mov byte ptr ss:[ebp-4],23
  00451986     .  8D85 98FCFFFF    lea eax,dword ptr ss:[ebp-368]
  0045198C     .  50               push eax
  0045198D     .  8B8D 6CFCFFFF    mov ecx,dword ptr ss:[ebp-394]
  00451993     .  81C1 AC000000    add ecx,0AC
  00451999     .  E8 42A5FFFF      call unpacked.0044BEE0
  0045199E     .  C645 FC 22       mov byte ptr ss:[ebp-4],22
  004519A2     .  8D8D 98FCFFFF    lea ecx,dword ptr ss:[ebp-368]
  004519A8     .  E8 13DBFCFF      call unpacked.0041F4C0
  004519AD     .  C645 FC 06       mov byte ptr ss:[ebp-4],6
  004519B1     .  8D8D 97FCFFFF    lea ecx,dword ptr ss:[ebp-369]
  004519B7     .  E8 D4330100      call unpacked.00464D90
  004519BC     .  EB 72            jmp short unpacked.00451A30
  004519BE     >  8D8D 7FFCFFFF    lea ecx,dword ptr ss:[ebp-381]
  004519C4     .  E8 A7270000      call unpacked.00454170
  004519C9     .  8985 E0FBFFFF    mov dword ptr ss:[ebp-420],eax
  004519CF     .  8B8D E0FBFFFF    mov ecx,dword ptr ss:[ebp-420]
  004519D5     .  898D DCFBFFFF    mov dword ptr ss:[ebp-424],ecx
  004519DB     .  C645 FC 24       mov byte ptr ss:[ebp-4],24
  004519DF     .  8B95 DCFBFFFF    mov edx,dword ptr ss:[ebp-424]
  004519E5     .  52               push edx                            ; /Arg2
  004519E6     .  68 387F4B00      push unpacked.004B7F38              ; |Arg1 = 004B7F38 ASCII "Please support shareware development by registering Rhino."
  004519EB     .  8D8D 80FCFFFF    lea ecx,dword ptr ss:[ebp-380]      ; |
  004519F1     .  E8 EA9EFDFF      call unpacked.0042B8E0             ; \unpacked.0042B8E0
  004519F6     .  C645 FC 25       mov byte ptr ss:[ebp-4],25

* * *
```

这样基本上有了个了解。

进入页面后，点 `About`，没看到注册画面。

在 `Rhino` 的网站上，我们看到：

"Buy `Rhino` online
Registrations are processed by `Shareit`, a well-known and reliable secure payment service.
After registering, you'll receive a personal key file, that can be used to enable all the program features"

原来是 keyfile 保护的。

本以为在 `unpacked.0042CA10` 里读注册文件，可是不是想象的那样。

那不可能不读写注册文件吧，于是 `bpx` `fopen`

运行程序，中断，啊~~~第一次读写的文件就是注册文件！

```
代码:

* * *

0042C510   /$  81EC 8C010000   sub esp,18C
0042C516   |.  53               push ebx
0042C517   |.  55               push ebp
0042C518   |.  56               push esi
0042C519   |.  8BB424 9C010000  mov esi,dword ptr ss:[esp+19C]
0042C520   |.  8BE9             mov ebp,ecx
0042C522   |.  57               push edi
0042C523   |.  33C0             xor eax,eax
0042C525   |.  B9 63000000      mov ecx,63
0042C52A   |.  8BFD             mov edi,ebp
0042C52C   |.  68 90694B00      push unpacked.004B6990           ; /mode = "rb"
0042C531   |.  F3:AB            rep stos dword ptr es:[edi]      ; |
0042C533   |.  56               push esi                          ; |path
0042C534   |.  89B5 8C010000    mov dword ptr ss:[ebp+18C],esi    ; |
0042C53A   |.  FF15 20A24900    call dword ptr ds:[<&MSVCRT.fopen>] ; \fopen

* * *
```

`//0012F48C   0202DFA0  ASCII "C:\Program Files\Rhino2\key"`



### 代码片段 1

```
0042C540   |.  8BD8            mov ebx,eax
0042C542   |.  83C4 08         add esp,8
0042C545   |.  85DB            test ebx,ebx
0042C547   |.  75 40           jnz short unpacked.0042C589
0042C549   |.  83C9 FF         or ecx,FFFFFFFF
0042C54C   |.  8BFE            mov edi,esi
0042C54E   |.  F2:AE           repne scas byte ptr es:[edi]
0042C550   |.  F7D1            not ecx
0042C552   |.  83C1 07         add ecx,7
0042C555   |.  51              push ecx
0042C556   |.  E8 C1500600     call <jmp.&MSVCRT.operator new>
0042C55B   |.  56              push esi                          ; /<%s> = "C:\Program Files\Rhino2\key"
0042C55C   |.  8BF8            mov edi,eax                       ; |
0042C55E   |.  68 88694B00     push unpacked.004B6988              ; |format = "%s.dat"
0042C563   |.  57              push edi                          ; |s
0042C564   |.  FF15 64A24900   call dword ptr ds:[<&MSVCRT.sprintf>]   ; \sprintf

0042C56A   |.  68 90694B00     push unpacked.004B6990                ; /mode = "rb"
0042C56F   |.  57              push edi                            ; |path = "C:\Program Files\Rhino2\key.dat"
0042C570   |.  FF15 20A24900   call dword ptr ds:[<&MSVCRT.fopen>]     ; \fopen

0042C576   |.  57              push edi
0042C577   |.  8BD8            mov ebx,eax
0042C579   |.  E8 72500600     call unpacked.004915F0
0042C57E   |.  83C4 1C         add esp,1C
0042C581   |.  85DB            test ebx,ebx
0042C583   |.  0F84 7C010000   je unpacked.0042C705
0042C589   |>  6A 02           push 2                            ; /whence = SEEK_END
0042C58B   |.  6A 00           push 0                            ; |offset = 0
0042C58D   |.  53              push ebx                          ; |stream = msvcrt.77C2FCE0
0042C58E   |.  FF15 0CA24900   call dword ptr ds:[<&MSVCRT.fseek>]    ; \fseek

0042C594   |.  53              push ebx                          ; /stream = msvcrt.77C2FCE0
0042C595   |.  FF15 10A24900   call dword ptr ds:[<&MSVCRT.ftell>]    ; \ftell
0042C59B   |.  83C4 10         add esp,10
0042C59E   |.  3D 8C010000     cmp eax,18C                       ;size(key.dat)=18C
0042C5A3   |.  0F85 B2000000   jnz unpacked.0042C65B
0042C5A9   |.  53              push ebx                          ; /stream = msvcrt.77C2FCE0
0042C5AA   |.  FF15 18A24900   call dword ptr ds:[<&MSVCRT.rewind>]    ; \rewind

0042C5B0   |.  53              push ebx                          ; /stream
0042C5B1   |.  68 8C010000     push 18C                          ; |n = 18C (396.)
0042C5B6   |.  8D4424 1C       lea eax,dword ptr ss:[esp+1C]     ; |
0042C5BA   |.  6A 01           push 1                            ; |size = 1
0042C5BC   |.  50              push eax                          ; |ptr = 0012F4A4
0042C5BD   |.  FF15 14A24900   call dword ptr ds:[<&MSVCRT.fread>]    ; \fread

//18C byte= 63 Dword


0042C5C3   |.  83C4 14         add esp,14
0042C5C6   |.  33FF            xor edi,edi
0042C5C8   |.  B8 78563412     mov eax,12345678
0042C5CD   |.  33C9            xor ecx,ecx
0042C5CF   |.  90              nop
0042C5D0   |>  8B548C 10       /mov edx,dword ptr ss:[esp+ecx*4+10]  ;esp+10=0012F4A4
0042C5D4   |.  8BF0            |mov esi,eax
0042C5D6   |.  0FAFF0          |imul esi,eax
0042C5D9   |.  03F8            |add edi,eax  ====\\
0042C5DB   |.  2BF2            |sub esi,edx   ===/exchanged
0042C5DD   |.  33FA            |xor edi,edx
0042C5DF   |.  41              |inc ecx
0042C5E0   |.  83F9 63         |cmp ecx,63                       ;Is the END?
0042C5E3   |.  8BC6            |mov eax,esi                      ;can be moved before 'inc ecx'
0042C5E5   |.^ 7C E9           \jl short unpacked.0042C5D0
0042C5E7   |.  33C9            xor ecx,ecx
0042C5E9   |.  3BF9            cmp edi,ecx                       ;edi=0?
0042C5EB   |.  75 37           jnz short unpacked.0042C624        ;Don't jump
```

`mov eax,12345678`

```
mov esi,12345678
xor edi,edi
for(i=0;i<63;i++)
{
    edx=base[i]
    //esi=eax
    //esi=esi*eax-edx
    esi=esi*esi-edx
    edi=(edi+esi) xor edx
    //eax=esi
}
```
after the loop,`edi` expects to be zero!
if the last Dword equal with the (`edi+esi`) ,then `edi` will be ZERO!
so we could do that easily!

### 代码片段 2

```
0042C5ED   |.  8B4424 10       mov eax,dword ptr ss:[esp+10]       ;the first Dword
0042C5F1   |.  B9 01000000     mov ecx,1
0042C5F6   |>  8B7C8C 10       /mov edi,dword ptr ss:[esp+ecx*4+10]
0042C5FA   |.  8D3401          |lea esi,dword ptr ds:[ecx+eax]     ;esi=ecx+eax
0042C5FD   |.  0FAFF0          |imul esi,eax                       ;esi=esi*eax
0042C600   |.  33F8            |xor edi,eax
0042C602   |.  897C8C 10       |mov dword ptr ss:[esp+ecx*4+10],edi    ;modify Dword
0042C606   |.  41              |inc ecx                            ;move to the next Dword
0042C607   |.  83F9 63         |cmp ecx,63
0042C60A   |.  8BD7            |mov edx,edi
0042C60C   |.  8D8416 78563412 |lea eax,dword ptr ds:[esi+edx+12345678] ;eax=esi+edx+12345678
0042C613   |.^ 7C E1           \jl short unpacked.0042C5F6
```

`eax=base[0];`
(`edi=0,implied`)

```
for(i=1;i<63;i++)
{
    edi=base[i];
    edi=edi xor eax;
    base[i]=edi;
    //esi=eax+i;
    //esi=esi*eax;
    //esi=(eax+i)*eax;
    //eax=esi+edi+12345678
    eax=(eax+i)*eax+edi+12345678
}
```

### 代码片段 3

```
0042C615   |.  B9 63000000     mov ecx,63
0042C61A   |.  8D7424 10       lea esi,dword ptr ss:[esp+10]        ;0012FAA4,Buffer pointer
0042C61E   |.  8BFD            mov edi,ebp                          ;0202C030,Buffer2 pointer
0042C620   |.  F3:A5           rep movs dword ptr es:[edi],dword ptr ds:[esi]
0042C622   |.  33C9            xor ecx,ecx
0042C624   |>  B8 B0C84900     mov eax,unpacked.0049C8B0
```

`[Data]`

```
0009c8b0h: 1A 00 00 00 02 45 F8 01 3E 72 C5 02 A6 2F F5 02 ; .....E?>r???
0009c8c0h: 2C 56 A9 01 B0 85 18 02 B6 9C AB 01 94 05 1C 04 ; ,V?皡..稖??..
0009c8d0h: 2A 09 BC 01 A7 2F F5 02 7A 85 A9 01 F4 91 0C 04 ; *.???z 叐.魬..
0009c8e0h: 5C 2E 2B 04 00 00 00 00                      ; \.+......
```

`[End Data]`

### 代码片段 4

```
0042C629   |.  8DA424 00000000 lea esp,dword ptr ss:[esp]
0042C630   |> /8B55 04         /mov edx,dword ptr ss:[ebp+4]     ;0202C030,the 2nd Dword
0042C633   |.|3B10             |cmp edx,dword ptr ds:[eax]
0042C635   |.|75 03           |jnz short unpacked.0042C63A
0042C637   |.|894D 04         |mov dword ptr ss:[ebp+4],ecx     ;set zero
0042C63A   |> |8B10           |mov edx,dword ptr ds:[eax]
0042C63C   |.|83C0 04         |add eax,4
0042C63F   |.|85D2           |test edx,edx
0042C641   |.^\75 ED          \jnz short unpacked.0042C630
```
if the first Dword=one of the 13 Dword,then,set the 2nd Dword to 0!

```
0042C643   |.  B8 E8C84900     mov eax,unpacked.0049C8E8
```

`[DATA]`

```
0009c8e8h: 2E 37 24 03 E4 54 C4 02 00 00 00 00            ; .7$.銽?.....
```

`[END DATA]`



# 代码段与逻辑分析

```
代码:

* * *

0042C648   |>  8B55 04          /mov edx,dword ptr ss:[ebp+4]
0042C64B   |.  3B10             |cmp edx,dword ptr ds:[eax]
0042C64D   |.  75 03            |jnz short unpacked.0042C652
0042C64F   |.  894D 04          |mov dword ptr ss:[ebp+4],ecx
0042C652   |>  8B10             |mov edx,dword ptr ds:[eax]
0042C654   |.  83C0 04          |add eax,4
0042C657   |.  85D2             |test edx,edx
0042C659   |.^ 75 ED            \jnz short unpacked.0042C648

* * *
```

if the first `Dword`=one of the 2 `Dword`,then,set the 1st `Dword` to 0!

```
代码:

* * *

0042C65B   |>  53               push ebx                                       ; /stream
0042C65C   |.  8B1D 28A24900   mov ebx,dword ptr ds:[<&MSVCRT.fclose>]        ; |msvcrt.fclose
0042C662   |.  FFD3            call ebx                                         ; \fclose

0042C664   |.  8B45 00         mov eax,dword ptr ss:[ebp]     ;ebp=0202C030
0042C667   |.  83C4 04         add esp,4
0042C66A   |.  40              inc eax                    ;eax++ *****
0042C66B   |.  8945 00         mov dword ptr ss:[ebp],eax     ;write back
0042C66E   |.  B9 63000000     mov ecx,63
0042C673   |.  8BF5            mov esi,ebp                ;0202C030
0042C675   |.  8D7C24 10       lea edi,dword ptr ss:[esp+10]  ;esp+10=0012F4A4
0042C679   |.  F3:A5           rep movs dword ptr es:[edi],dword ptr ds:[esi]
0042C67B   |.  8B4424 10       mov eax,dword ptr ss:[esp+10]  ;esp+10=0012F4A4

0042C67F   |.  B9 01000000     mov ecx,1
0042C684   |>  8B548C 10       /mov edx,dword ptr ss:[esp+ecx*4+10]
0042C688   |.  8BF2            |mov esi,edx
0042C68A   |.  33F0            |xor esi,eax
0042C68C   |.  89748C 10       |mov dword ptr ss:[esp+ecx*4+10],esi
0042C690   |.  8D3401          |lea esi,dword ptr ds:[ecx+eax]
0042C693   |.  0FAFF0         |imul esi,eax
0042C696   |.  41              |inc ecx
0042C697   |.  83F9 63         |cmp ecx,63
0042C69A   |.  8D8416 78563412 |lea eax,dword ptr ds:[esi+edx+12345678]
0042C6A1   |.^ 7C E1           \jl short unpacked.0042C684

* * *
```

`eax=base[0]`

```
for(i=1;i<63;i++)
{
    edx=base[i];
    esi=edx xor eax;
    base[i]=esi;
    esi=(eax+i)*eax;
    eax=esi+edx+12345678;
}
```

same as the block `0042C5ED-0042C613`.

```
代码:

* * *

0042C6A3   |.  33C9            xor ecx,ecx
0042C6A5   |.  898C24 98010000 mov dword ptr ss:[esp+198],ecx   ;esp+198=0012F62C
0042C6AC   |.  33F6            xor esi,esi
0042C6AE   |.  B8 78563412     mov eax,12345678
0042C6B3   |> /8B548C 10       /mov edx,dword ptr ss:[esp+ecx*4+10]
0042C6B7   |.|8BF8             |mov edi,eax
0042C6B9   |.|0FAFF8          |imul edi,eax
0042C6BC   |.|03F0            |add esi,eax
0042C6BE   |.|2BFA            |sub edi,edx
0042C6C0   |.|33F2            |xor esi,edx
0042C6C2   |.|41              |inc ecx
0042C6C3   |.|83F9 63         |cmp ecx,63
0042C6C6   |.|8BC7            |mov eax,edi
0042C6C8   |.\7C E9           \jl short unpacked.0042C6B3

* * *
```

`esi=0`

`eax=0x12345678`

```
for(i=0;i<63;i++)
{
    edx=base[i]
    //edi=eax
    //edi=edi*eax
    //esi=esi+eax
    //esi=esi xor edx
    esi=(esi+eax)xor edx
    //edi=eax*eax
    //edi=edi-edx
    //edi=eax*eax-edx
    //eax=edi
    eax=eax*eax-edx
}
```

```
代码:

* * *

0042C6CA   |.  8B85 8C010000   mov eax,dword ptr ss:[ebp+18C]   ;EAX 0202DFA0 ASCII "C:\Program Files\Rhino2\key"
0042C6D0   |.  68 D00B4B00     push unpacked.004B0BD0           ; /mode = "wb+"
0042C6D5   |.  50              push eax                         ; |path
0042C6D6   |.  89B424 A0010000 mov dword ptr ss:[esp+1A0],esi          ; |esp+1A0=0012F62C
0042C6DD   |.  FF15 20A24900   call dword ptr ds:[<&MSVCRT.fopen>]       ; \fopen
0042C6E3   |.  8BF0            mov esi,eax
0042C6E5   |.  83C4 08         add esp,8
0042C6E8   |.  85F6            test esi,esi
0042C6EA   |.  74 19           je short unpacked.0042C705
0042C6EC   |.  56              push esi                         ; /stream
0042C6ED   |.  68 8C010000     push 18C                         ; |n = 18C (396.)
0042C6F2   |.  8D4C24 18       lea ecx,dword ptr ss:[esp+18]            ; |
0042C6F6   |.  6A 01            push 1                         ; |size = 1
0042C6F8   |.  51              push ecx                       ; |ptr = 0012F4A4
0042C6F9   |.  FF15 1CA24900   call dword ptr ds:[<&MSVCRT.fwrite>]     ; \fwrite
0042C6FF   |.  56              push esi
0042C700   |.  FFD3            call ebx
0042C702   |.  83C4 14         add esp,14
0042C705   |>  5F              pop edi
0042C706   |.  5E              pop esi
0042C707   |.  8BC5            mov eax,ebp
0042C709   |.  5D              pop ebp
0042C70A   |.  5B              pop ebx
0042C70B   |.  81C4 8C010000   add esp,18C
0042C711   \.  C2 0400         retn 4

* * *
```

===

```
代码:

* * *

0042CA10     $  53                push ebx
0042CA11     .  56                push esi
0042CA12     .  8BF1             mov esi,ecx
0042CA14     .  53                push ebx
0042CA15     .  E8 01000000      call unpacked.0042CA1B
0042CA1A        81                db 81
0042CA1B     .  C3                retn
0042CA1C     ?  FF88 FFFF5B8B    dec dword ptr ds:[eax+8B5BFFFF]

* * *
```

\*\*\*\*\*\*注意，上面这段代码反汇编错误，上面的机器码 `5B,8B`，就是下面反汇编出来的机器码。

\*\*\*\*\*\*不知怎么回事...迷惑...

找了一块由`"9090"`的区域，填入机器码 `81 C3 FF 88 FF 88`，发现是 `add ebx,FFFF88FF` 语句！

`00377DB0+FFFF88FF=003706AF`，实际执行时，`ebx=003706AF`,这说明确实是执行了 `add ebx,FFFF88FF`，

那么，上面的那个 `call unpacked.0042CA1B` 语句又怎么执行呢？

程序在执行到 `call unpacked.0042CA1B` 时，然后 `EIP=0042CA1B`，`Retn` 后，`EIP=0042CA1A`,接着就执行了

`add ebx,FFFF88FF`，接下来 `pop ebx,`这说明 `0042CA14-0042CA20` 间的语句是没有用的！

完全可以 `NOP` 掉！

```
代码:

* * *

0042CA20        5B               pop ebx
0042CA21        8B56 04          mov edx,dword ptr ds:[esi+4];esi=0202C030,2nd Dword
0042CA24     .  85D2             test edx,edx
0042CA26     .  74 15            je short unpacked.0042CA3D
0042CA28     .  33C0             xor eax,eax
0042CA2A     .  8D8E 88000000    lea ecx,dword ptr ds:[esi+88]
0042CA30     >  3911             cmp dword ptr ds:[ecx],edx
0042CA32     .  74 13            je short unpacked.0042CA47
0042CA34     .  40               inc eax
0042CA35     .  83C1 04          add ecx,4
0042CA38     .  83F8 40          cmp eax,40
0042CA3B     .^ 7C F3            jl short unpacked.0042CA30
0042CA3D        33C0             xor eax,eax
0042CA3F        85D2             test edx,edx
0042CA41        0F95C0            setne al                 ；Set flag
0042CA44     .  5E               pop esi
0042CA45     .  5B               pop ebx
0042CA46     .  C3               retn

* * *
```

=================

## 关于用户名的显示问题



### 逆向分析与异常定位

```assembly
004510D8     .  50             push eax                               ; userName here
004510D9       8B             db 8B
004510DA       15             db 15
004510DB     .  C07D 4B 00    sar byte ptr ss:[ebp+4B],0
004510DF     .  52             push edx                               ; |format
004510E0     .  8D85 4CFEFFFF lea eax,dword ptr ss:[ebp-1B4]          ; |
004510E6     .  50             push eax                               ; |s
004510E7     .  FF15 64A24900  call dword ptr ds:[<&MSVCRT.sprintf>] ; \sprintf

0012F3EC   0012F66C  |s = 0012F66C
0012F3F0   004B7DC4  |format = "Registered to: %s"
0012F3F4   0202C038  \<%s> = "&3:|?|\`\[??/o?&
```

出现异常，关键指令序列如下：

```assembly
004562B2    |> \8BD1          mov edx,ecx
004562B4    |.  C1E9 02        shr ecx,2
004562B7    |.  8BF0          mov esi,eax
004562B9    |.  8BFB          mov edi,ebx
004562BB    |.  8BC7          mov eax,edi
004562BD    |.  F3:A5         rep movs dword ptr es:[edi],dword ptr ds:[esi]
```

上面这一句出现异常，`esi`=67FED074，`edi`=0。

完整上下文：

```assembly
004562B2    |> \8BD1          mov edx,ecx
004562B4    |.  C1E9 02        shr ecx,2
004562B7    |.  8BF0          mov esi,eax
004562B9    |.  8BFB          mov edi,ebx
004562BB    |.  8BC7          mov eax,edi
004562BD    |.  F3:A5         rep movs dword ptr es:[edi],dword ptr ds:[esi]
004562BF    |.  8BCA          mov ecx,edx
004562C1    |.  83E1 03       and ecx,3
004562C4    |.  F3:A4         rep movs byte ptr es:[edi],byte ptr ds:[esi]
004562C6    |.  034424 14     add eax,dword ptr ss:[esp+14]
004562CA    |.  8945 00       mov dword ptr ss:[ebp],eax
004562CD    |.  8B4C24 20     mov ecx,dword ptr ss:[esp+20]
004562D1    |.  5F             pop edi
004562D2    |.  5E             pop esi
004562D3    |.  5D             pop ebp
004562D4    |.  5B             pop ebx
004562D5    |.  64:890D 00000000 mov dword ptr fs:[0],ecx
004562DC    |.  83C4 1C       add esp,1C
004562DF    \.  C3             retn
```

### 修复与算法推导

猜测是用户名显示不正确，于是修改`0202C038`所指向的字符串，修改后无异常。

修改的规则是字符串必须为可打印符(`20`-`7E`，但经过试验，不支持中文，没深入考虑汉字注册问题...)，必须有字符串结束符`00`。

有了这些基础，下面就得寻找用户名的可逆算法。

#### 算法总结

我们来回顾一下，将`key.dat`文件读到缓冲区以后：

1.  程序段`0042C5C3`-`0042C5E5`来检验数据。
2.  取出第一个双字存入`eax`，`ecx`初值为`1`，然后按照某个算法修正后面的双字。

算法描述：
```
for(ecx=1; ecx<63; ecx++)
{
    esi = (ecx + eax) * eax;
    取出下一个双字与 eax 存入 edi，与 eax 进行 Xor 后写回；
    修改 eax, eax = esi + edi + 12345678
}
```

我要显示的用户名是`"winndy"`，在内存中是这样：
`77 69 6E 6E 64 79 00 00` -> `winndy..`

这是第`3`和第`4`个双字。
我们设第一，二个双字为`Dword1`，`Dword2`。

```
New_Dword2 = Dword2 xor Dword1;
eax = (Dword1 + 1) * Dword1 + New_Dword2 + 0x12345678

New_Dword3 = Dword3 xor eax;  //make sure New_Dword3=6E6E6977

New_eax = (eax + 2) * eax + New_Dword3 + 0x12345678

New_Dword4 = Dword4 xor New_eax  //make sure New_Dword4=00007964

New_eax = (eax + 3) * eax + New_Dword4 + 0x12345678

New_Dword5 = Dword5 xor New_eax  //make sure New_Dword5=00000000
```

`New_Dword2`不能为下面的双字中的一个：
1.  `0000001A`
2.  `01F84502`
3.  `02C5723E`
4.  `02F52FA6`
5.  `01A9562C`
6.  `021885B0`
7.  `01AB9CB6`
8.  `041C0594`
9.  `01BC092A`
10. `02F52FA7`
11. `01A9857A`
12. `040C91F4`
13. `042B2E5C`
14. `0324372E`
15. `02C454E4`

数据参考：
```
[Data]
0009c8b0h: 1A 00 00 00 02 45 F8 01 3E 72 C5 02 A6 2F F5 02 ; .....E?>r???
0009c8c0h: 2C 56 A9 01 B0 85 18 02 B6 9C AB 01 94 05 1C 04 ; ,V?皡..稖??..
0009c8d0h: 2A 09 BC 01 A7 2F F5 02 7A 85 A9 01 F4 91 0C 04 ; \*.???z 叐.魬..
0009c8e0h: 5C 2E 2B 04 00 00 00 00                      ; \\.+......
0009c8e8h: 2E 37 24 03 E4 54 C4 02 00 00 00 00          ; .7$.銽?.....
[End Data]
```

推导逻辑：
1.  可以先定`Dword1`，然后与上面的`15`个双字`xor`，`Dword2`不取`15`个双字中的任何一个。
2.  这样定下了`Dword2`。
3.  由`New_Dword2=Dword2 xor Dword1`求得`New_Dword2`。
4.  再由`eax=(Dword1+1)*Dword1+New_Dword2+0x12345678`求得`eax`。
5.  然后`Dword3=New_Dword3 xor eax`，而`New_Dword3=6E6E6977`，这是用户名，这样就求得了`Dword3`。
6.  `eax=(eax+2)*eax+New_Dword3+0x12345678`。
7.  再`Dword4=New_Dword4 xor eax`，而`New_Dword4=00007964`。
8.  `eax=(eax+2)*eax+New_Dword4+0x12345678`。
9.  `Dword5=New_Dword5 xor eax`。
10. 我们要`make sure` `New_Dword5=00000000`，也就是说将`eax`取反。
11. 这里为了方便，我们不管用户名的长度为多少，总是在用户名最后的双字后面加一个为`0`的双字(当然，必须有空间)。
12. 当用户名长度不是`4`的整数倍时，我们在不足的`byte`上也是填充`0`。

这样，就可以写出一个注册机了，生成`key.dat`。在读取注册文件`key.dat`之后，生成了一个`key`文件，再次运行直接读`key`文件。

这个程序求逆还真伤脑筋，用`asm`写注册机的时候，调试花了我大大半天的时间，哎！终于搞定了！

### 其他信息

*   **Greetings**: 看雪论坛, FCG 论坛, DFCG 论坛等。
*   **说明**: 注册机采用`TASM`写，采用`jB`的模版，感谢！
*   **完稿时间**: 2005.05.20，16:05，天气:多云,广州
*   **KeyFile Generator Source**: 附件:`Rhnio_keygen_Source.rar`
*   文件哈希: `4dd466d0e1e0cc123afbe504da2462ca`
