
# 趁热打铁[申请加入 FCG 之四]Talisman V2.81 分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-194.htm`](https://www.pediy.com/kssd/pediy07/pediy7-194.htm)

Talisman V2.81 注册算法分析

【破解作者】 winndy

【作者邮箱】 CNwinndy@hotmail.com

【使用工具】 PEID v0.93 OllyDbg v1.10 fly 修改版 DeDe.3.50.04.1635

【破解平台】 Winxp SP2

【软件名称】 Talisman Desktop 2.81 (build 2810)

【官方网址】 http://www.lighttek.com

【编写语言】 Borland Delphi 4.0 - 5.0

【破解声明】 For Study ,For Fun,

【破解说明】 无壳,失误之处还望指出

【破解过程】 PEID：Borland Delphi 4.0 - 5.0。

DeDe：反汇编。找到 About 窗口，`ClassName` 是 `TAboutBox`，双击，打开 form 看看，DeDE 报错，另外再想办法确定输入注册信息后点击的那个按钮吧。

点 Procedures，选择 `UnitName` 为 about，右边的 event 中有 `OKButtonClick`，`SpeedButton4Click`，`SpeedButton1Click`，通过观察，可知 `SpeedButton1Click` 事件对应着输入注册码后所按的按钮。于是用 OD 载入，`ctrl+g`，`00489435`，在 `SpeedButton1Click` 的开始处下断。

下面是从 DeDe 中 Copy 而来的代码，不过要注意，DeDe 反汇编出来的不总是对的，可能还有点小 Bug，后面会发现。

注册时：首先，输 name：CNwinndy，code：1234567890；第二遍跟踪的时候 code：12345678901234567890。后面会看到为什么。

```
12:11 2005-4-29   55                     push    ebp
00489435   8BEC                 mov     ebp, esp
00489437   6A00                 push    $00
00489439   6A00                 push    $00
0048943B   6A00                 push    $00
0048943D   53                   push    ebx
0048943E   56                   push    esi
0048943F   8BD8                 mov     ebx, eax
00489441   33C0                 xor     eax, eax
00489443   55                   push    ebp
* Possible String Reference to: '閥_?豚^[嬪]?'
|
00489444   6866954800           push    $00489566
***** TRY
|
00489449   64FF30               push    dword ptr fs:[eax]
0048944C   648920               mov     fs:[eax], esp
0048944F   8D55FC               lea     edx, [ebp-$04]
* Reference to control TAboutBox.edit_code : TEdit     //这里是取注册码啦
|
00489452   8B83E0020000         mov     eax, [ebx+$02E0]
* Reference to: controls.TControl.GetText(TControl):TCaption;
|
00489458   E8B319FAFF           call    0042AE10
0048945D   8D55F8               lea     edx, [ebp-$08]
* Reference to control TAboutBox.edit_code : TEdit
|
00489460   8B83E0020000         mov     eax, [ebx+$02E0]
* Reference to: controls.TControl.GetText(TControl):TCaption;
|
00489466   E8A519FAFF           call    0042AE10
0048946B   837DF800             cmp     dword ptr [ebp-$08], +$00
//D [EBP-8]
//0101B708  31 32 33 34 35 36 37 38  12345678
//0101B710  39 30 00 20               90.
0048946F   0F84CB000000         jz      00489540
00489475   837DFC00             cmp     dword ptr [ebp-$04], +$00
//D [EBP-8]
//0101B6F0  31 32 33 34 35 36 37 38  12345678
//0101B6F8  39 30 00 00               90..
00489479   0F84C1000000         jz      00489540
0048947F   8B45FC               mov     eax, [ebp-$04]   //EAX 0101B6F0 ASCII "1234567890"
* Reference to: System.Proc_00403DBC
|
00489482   E835A9F7FF           call    00403DBC          //跟进，
  {
    00403DBC   /$  85C0               test eax,eax                  ；注册码是否为空
    00403DBE   |.  74 03              je short talisman.00403DC3
    00403DC0   |.  8B40 FC            mov eax,dword ptr ds:[eax-4]  ；EAX=0000000A，注册码长度
    00403DC3   \>  C3                 retn
  }
00489487   8BF0                 mov     esi, eax                   ；保存注册码长度
00489489   85F6                 test    esi, esi
0048948B   7E39                 jle     004894C6
0048948D   B801000000           mov     eax, $00000001             ；计数器初值为 1
LOOP：
00489492   8B55FC               mov     edx, [ebp-$04]             ；EDX 0101B6F0 ASCII "1234567890"
00489495   8A5402FF             mov     dl, byte ptr [edx+eax-$01] ；取注册码第 eax 个字符
00489499   80FA39               cmp     dl, $39
0048949C   7708                 jnbe    004894A6                   ；大于 9 就跳
0048949E   8B4DFC               mov     ecx, [ebp-$04]             ；ECX 0101B6F0 ASCII "1234567890"
004894A1   80FA30               cmp     dl, $30
004894A4   731C                 jnb     004894C2                   ；不小于 0 就跳
004894A6   B201                 mov     dl, $01
* Reference to control TAboutBox.Panel2 : TPanel
|
004894A8   8B83FC020000         mov     eax, [ebx+$02FC]
* Reference to: controls.TControl.SetVisible(TControl;Boolean);     ；
|
004894AE   E87518FAFF           call    0042AD28
004894B3   B201                 mov     dl, $01
* Reference to control TAboutBox.Timer1 : TTimer
|
004894B5   8B8300030000         mov     eax, [ebx+$0300]
* Reference to: extctrls.TTimer.SetEnabled(TTimer;Boolean);
|
004894BB   E86818FCFF           call    0044AD28
004894C0   EB7E                 jmp     00489540
004894C2   40                   inc     eax
004894C3   4E                   dec     esi
004894C4   75CC                 jnz     00489492           //GOTO LOOP
004894C6   8BC3                 mov     eax, ebx
* Reference to : TAboutBox.Proc_0048961C()
|
004894C8   E84F010000           call    0048961C        *******       //见[分析一]
* Reference to: Unit_004AC9A4.Proc_004AD8D0
|
004894CD   E8FE430200           call    004AD8D0        ********    //见[分析二]  这两个 call 很重要
004894D2   84C0                 test    al, al                    //al=01，则注册成功
004894D4   7450                 jz      00489526
004894D6   BA2C010000           mov     edx, $0000012C
* Reference to AboutBox
|
004894DB   A154494D00           mov     eax, dword ptr [$004D4954]
* Reference to: controls.TControl.SetWidth(TControl;Integer);
|
004894E0   E85B11FAFF           call    0042A640                  //F8
* Reference to pointer to GlobalVar_004D4D90
|
004894E5   A1BC354C00           mov     eax, dword ptr [$004C35BC]
004894EA   FF30                 push    dword ptr [eax]
004894EC   687C954800           push    $0048957C
* Reference to pointer to GlobalVar_004D4AA8
|
004894F1   A1A8314C00           mov     eax, dword ptr [$004C31A8]
004894F6   FF30                 push    dword ptr [eax]
004894F8   8D45F4               lea     eax, [ebp-$0C]
004894FB   BA03000000           mov     edx, $00000003
* Reference to: System.Proc_00403E7C
|
00489500   E877A9F7FF           call    00403E7C                  //F8
00489505   8B55F4               mov     edx, [ebp-$0C]            //EDX 0101B764 ASCII "Registered for CNwinndy"
                                                     //看到这里，松口气了，
                                                     //剩下的代码可以 F9，GO！
                                                     //出现了注册成功画面，
                                                     //右边的注册面板则消失了
                                                     //点 OK，再 F9，点它的"START"button
                                                     ，那个'关于和注册'消失！
                                                     //再打开，还是没注册。没有写进注册表。
* Reference to AboutBox
|
00489508   A154494D00           mov     eax, dword ptr [$004D4954]
* Reference to control Label_user : TLabel
|
0048950D   8B80E8020000         mov     eax, [eax+$02E8]
* Reference to: controls.TControl.SetText(TControl;TCaption);
|
00489513   E82819FAFF           call    0042AE40
* Reference to TForm1 instance
|
00489518   A11C364C00           mov     eax, dword ptr [$004C361C]
0048951D   8B00                 mov     eax, [eax]
0048951F   33D2                 xor     edx, edx
* Reference to field TForm1.Tag : Longint
|
00489521   89500C               mov     [eax+$0C], edx
00489524   EB1A                 jmp     00489540
00489526   B201                 mov     dl, $01
```



\* Reference to control `TAboutBox.Panel2` : `TPanel`

```
|

00489528   8B83FC020000          mov     eax, [ebx+$02FC]

\* Reference to: `controls.TControl.SetVisible(TControl;Boolean);`

|

0048952E   E8F517FAFF            call    0042AD28

00489533   B201                 mov     dl, $01

\* Reference to control `TAboutBox.Timer1` : `TTimer`

|

00489535   8B8300030000          mov     eax, [ebx+$0300]

\* Reference to: `extctrls.TTimer.SetEnabled(TTimer;Boolean);`

|

0048953B   E8E817FCFF            call    0044AD28

00489540   33C0                 xor     eax, eax

00489542   5A                   pop     edx

00489543   59                   pop     ecx

00489544   59                   pop     ecx

00489545   648910               mov     fs:[eax], edx

\*\*\*\*\*\* FINALLY

|

\* Possible String Reference to: '^\[嬪\]?

|

00489548   686D954800            push    $0048956D

0048954D   8D45F4               lea     eax, [ebp-$0C]

\* Reference to: `System.Proc_00403B40`

|

00489550   E8EBA5F7FF            call    00403B40

00489555   8D45F8               lea     eax, [ebp-$08]

\* Reference to: `System.Proc_00403B40`

|

00489558   E8E3A5F7FF            call    00403B40

0048955D   8D45FC               lea     eax, [ebp-$04]

\* Reference to: `System.Proc_00403B40`

|

00489560   E8DBA5F7FF            call    00403B40

00489565   C3                   ret

\* Reference to: `System.Proc_004035E4`

|

00489566   E979A0F7FF            jmp     004035E4

0048956B   EBE0                 jmp     0048954D

\*\*\*\*\*\* END

|

0048956D   5E                   pop     esi

0048956E   5B                   pop     ebx

0048956F   8BE5                 mov     esp, ebp

00489571   5D                   pop     ebp

00489572   C3                   ret
```

`=====================================================================`

`[分析一]`

```
0048961C   55                   push    ebp

0048961D   8BEC                 mov     ebp, esp

0048961F   81C4E0FEFFFF         add     esp, $FFFFFEE0

00489625   53                   push    ebx

00489626   56                   push    esi

00489627   57                   push    edi

00489628   33D2                 xor     edx, edx

0048962A   8995E0FEFFFF         mov     [ebp+$FFFFFEE0], edx    //ebp-120=0012FAAC

00489630   8995E8FEFFFF         mov     [ebp+$FFFFFEE8], edx    //ebp-118

00489636   8995E4FEFFFF         mov     [ebp+$FFFFFEE4], edx    //ebp-11c

0048963C   8955F8               mov     [ebp-$08], edx

0048963F   8955F4               mov     [ebp-$0C], edx          //ebp-0c=0012FBC0

00489642   33C0                 xor     eax, eax

00489644   55                   push    ebp

\* Possible String Reference to: '閥漭胄\_^\[嬪\]?

|

00489645   685F984800            push    $0048985F

\*\*\*\*\* TRY

|

0048964A   64FF30                push    dword ptr fs:[eax]

0048964D   648920                mov     fs:[eax], esp

00489650   B201                 mov     dl, $01

\* Reference to class `TRegistry`

|

00489652   A158EE4400            mov     eax, dword ptr [$0044EE58]

\* Reference to: `Unit_0044EDF8.Proc_0044EF98`

|

00489657   E83C59FCFF            call    0044EF98        //F8

0048965C   8945FC               mov     [ebp-$04], eax

0048965F   BA01000080           mov     edx, $80000001

00489664   8B45FC               mov     eax, [ebp-$04]

\* Reference to: `Unit_0044EDF8.Proc_0044F030`

|

00489667   E8C459FCFF            call    0044F030

0048966C   B101                 mov     cl, $01

\* Possible String Reference to: '\\Software\\Microsoft\\Windows\\Current

|                                   Version\\Explorer\\Advanced'          //注意，

|

0048966E   BA78984800            mov     edx, $00489878

00489673   8B45FC               mov     eax, [ebp-$04]

\* Reference to: `Unit_0044EDF8.Proc_0044F18C`

|

00489676   E8115BFCFF            call    0044F18C

0048967B   6800010000            push    $00000100

00489680   8D8DEEFEFFFF         lea     ecx, [ebp+$FFFFFEEE]

\* Possible String Reference to: 'WCID'                                        //注意

|

00489686   BAC0984800            mov     edx, $004898C0

0048968B   8B45FC               mov     eax, [ebp-$04]

\* Reference to: `Unit_0044EDF8.Proc_0044F898`

|

0048968E   E80562FCFF            call    0044F898

00489693   0FB6BD25FFFFFF       movzx   edi, byte ptr [ebp+$FFFFFF25]  //00000014

注意，后面会再 store 到其他地方

0048969A   33C0                 xor     eax, eax

0048969C   8A8527FFFFFF         mov     al, byte ptr [ebp+$FFFFFF27]   //05

004896A2   668945F2             mov     [ebp-$0E], ax                 //ebp-0e=0012FBBE

004896A6   33C0                 xor     eax, eax

004896A8   8A852AFFFFFF         mov     al, byte ptr [ebp+$FFFFFF2A]   //04

004896AE   668945F0             mov     [ebp-$10], ax                 //ebp-10=0012FBBC

004896B2   33C0                 xor     eax, eax

004896B4   8A852DFFFFFF         mov     al, byte ptr [ebp+$FFFFFF2D]   //1D

004896BA   668945EE             mov     [ebp-$12], ax                 //ebp-12=0012FBBA

004896BE   BB00010000           mov     ebx, $00000100

004896C3   8DB5EEFEFFFF         lea     esi, [ebp+$FFFFFEEE]          //esi=0012FABA

004896C9   B8FF000000           mov     eax, $000000FF

\* Reference to: `system.@RandInt;`

|

004896CE   E8B995F7FF            call    00402C8C          //这段 call 的功能是初始化注册信
//这段 call 的功能是初始化注册信息数组，byte reginfo[0x100]

   {00402C8C    /$  6915 44404C00 05840808 imul edx,dword ptr ds:[4C4044],8088405；[4c4044]
=1EF18248；1EF18248\*8088405= 00F89393 7424AB68(EDX)

    00402C96    |.  42                  inc edx

    00402C97    |.  8915 44404C00       mov dword ptr ds:[4C4044],edx

    00402C9D    |.  F7E2                mul edx

    00402C9F    |.  89D0                mov eax,edx

    00402CA1    \\.  C3                  retn

   }

004896D3   8806                 mov     [esi], al

004896D5   46                   inc     esi

004896D6   4B                   dec     ebx

004896D7   75F0                 jnz     004896C9

//上面这个 loop 从地址 0012FABA 处开始初始化 100 个 byte，一共 32 行

//0012FABA  50 72 09 07 12 A6 1D 5F  Pr.?\_

0012FAC2  43 F8 C7 FB F7 F7 4E 91  C鱊

0012FACA  36 0A E9 C7 E8 99 A3 87  6.榍铏

0012FAD2  07 5E 16 B3 DF 28 90 B1  ^尺(惐

0012FADA  8F B0 4E 83 0A 3A C7 42  彴 N?:荁

0012FAE2  50 09 55 3D B0 10 9E 20  P.U=??

0012FAEA  25 4A 97 C4 3B 94 FD AD  %J 椖;旪

0012FAF2  0E 1D A2 43 40 A3 E6 51  @ｆQ

0012FAFA  06 24 37 5F A2 AD 84 DE  $7\_勣

0012FB02  ED 7F DF 4C C3 A2 D4 C3  ?週芒悦

0012FB0A  7C A5 1E 9C BD 2F D3 0C  |?溄/?

0012FB12  4E 85 26 F0 BA 4A 48 32  N?鸷 JH2

0012FB1A  F7 FE 2A 6C 4D 15 0F C0  齄\*lM

0012FB22  2F A5 34 06 E2 06 04 C2  /??

0012FB2A  02 D9 95 98 44 65 72 09  贂楧 er.

0012FB32  1F 29 DE D2 24 1D 1B 36  )抟$6

0012FB3A  2D 09 6B ED BC D6 C5 A6  -.k 砑峙

0012FB42  2F D8 80 3B 83 65 65 1A  /貈;僥 e

0012FB4A  01 39 F4 75 E3 87 CB 3C  9 魎銍?

0012FB52  DC AF 6D F3 13 F0 EA F3  墀 m?痍

0012FB5A  F0 9F 28 9C EB AB AE 82  馃(滊

0012FB62  09 8A 5B B0 DF C0 57 77  .奫斑繵 w

0012FB6A  46 A9 1A 67 F7 3A 74 72  F?g?tr

0012FB72  E3 D3 D8 54 E3 70 66 B2  阌豑鉷 f

0012FB7A  13 B0 6F A3 15 A2 6F 7B  皁?{

0012FB82  DE 3D BA 20 F3 EA 58 40  ??箨 X@

0012FB8A  23 A4 C7 13 1F 76 A7 AC  #でvК

0012FB92  99 65 8B E6 B4 17 5F 72  檈嬫?\_r

0012FB9A  E8 C1 33 A2 B8 1E 54 EA  枇 3⒏T

0012FBA2  D4 8B FB 37 82 8B 0F 8A  詪?倠

0012FBAA  6F 39 A0 93 3A 87 42 1F  o9\_?嘊

0012FBB2  63 44 53 91 6A 38 CC E2  cDS 慾 8 题
```



# 汇编代码分析

## 004896D9 至 004896FF

```assembly
004896D9   8D55F4           lea     edx, [ebp-$0C]          //0012FBC0
* Reference to AboutBox

004896DC   A154494D00       mov     eax, dword ptr [$004D4954]
* Reference to control edit_code : N.A.                     //获得 code

004896E1   8B80E0020000     mov     eax, [eax+$02E0]
* Reference to: controls.TControl.GetText(TControl):TCaption;

004896E7   E82417FAFF       call    0042AE10

004896EC   837DF400         cmp     dword ptr [ebp-$0C], +$00

004896F0   740D             jz      004896FF

004896F2   8B45F4           mov     eax, [ebp-$0C]   ；EAX 0101B710 ASCII "1234567890"
* Reference to: System.Proc_00403DBC

004896F5   E8C2A6F7FF       call    00403DBC

004896FA   83F814           cmp     eax, +$14              ；eax=0000000A，注册码长度
，这里很关键！！！！
；后面会看到，注册码长度必须为 20

004896FD   7D38             jnl     00489737               ：关键，要跳才行，输
入"1234567890"没跳

004896FF   6A07             push    $07                    //注意常数
00489701   68D7A1BD37       push    $37BDA1D7              //注意常数
```

## 00489706 调用 004082F8

```assembly
00489706   8D85E8FEFFFF     lea     eax, [ebp+$FFFFFEE8]
* Reference to: Unit_004073B8.Proc_004082F8

0048970C   E8E7EBF7FF       call    004082F8
   {
     004082F8   55                   push    ebp
     004082F9   8BEC                 mov     ebp, esp
     004082FB   83C4F8               add     esp, -$08
     004082FE   6A00                 push    $00
     00408300   8D5508               lea     edx, [ebp+$08]
；D EDX
；0012FA8C D7 A1 BD 37 07 00 00 00 住?... 这是压入的参数

     00408303   8955F8               mov     [ebp-$08], edx
     00408306   C645FC10             mov     byte ptr [ebp-$04], $10
     0040830A   8D4DF8               lea     ecx, [ebp-$08]
     * Possible String Reference to: '%d'
     |
     0040830D   BA28834000           mov     edx, $00408328
     * Reference to: Unit_004073B8.Proc_00409030
     |
     00408312   E8190D0000           call    00409030      ；进去，但没观察到什么，可 F8
     00408317   59                   pop     ecx
     00408318   59                   pop     ecx
     00408319   5D                   pop     ebp
     0040831A   C20800               ret     $0008
   }
```

## 00489711 参数合并与转换

```assembly
00489711   8B85E8FEFFFF     mov     eax, [ebp+$FFFFFEE8]   ；EAX 0101B728 ASCII
"30999945687"，这里有东东了
 ；运行 calc，输入十进制 30999945687(因为上面那个 call 中有格式串"%d")，转化为十六进制，737BDA1D7
，呵呵，知道了，刚压入了两个参数的，
 ；两个双字合并为 64 位的字，再转化为十进制了

00489717   50               push    eax
00489718   8D95E4FEFFFF     lea     edx, [ebp+$FFFFFEE4]
0048971E   B821771906       mov     eax, $06197721         //注意常数
* Reference to: Unit_004073B8.Proc_004082C8

00489723   E8A0EBF7FF       call    004082C8
     {
      004082C8   83C4F8               add     esp, -$08
      004082CB   6A00                 push    $00
      004082CD   89442404             mov     [esp+$04], eax
      004082D1   C644240800           mov     byte ptr [esp+$08], $00
      004082D6   8D4C2404             lea     ecx, [esp+$04]
      004082DA   8BC2                 mov     eax, edx
      * Possible String Reference to: '%d'
      |
      004082DC   BAF4824000           mov     edx, $004082F4
      * Reference to: Unit_004073B8.Proc_00409030
      |
      004082E1   E84A0D0000           call    00409030   ；还是上面转化 64 进制到 10 进制中的那
个 call
      004082E6   59                   pop     ecx
      004082E7   5A                   pop     edx
      004082E8   C3                   ret
     }

00489728   8B95E4FEFFFF     mov     edx, [ebp+$FFFFFEE4]   ；EDX 0101B740 ASCII
"102332193" 102332193=$06197721
0048972E   8D45F4           lea     eax, [ebp-$0C]
00489731   59               pop     ecx
；EAX 0012FBC0 D [eax] 0101B710 31 32 33 34 35 36 37 38 12345678
                         0101B718 39 30 00 90.
；ECX 0101B728 ASCII "30999945687"
；EDX 0101B740 ASCII "102332193"
* Reference to: System.Proc_00403E08
|
00489732   E8D1A6F7FF       call    00403E08              //这个 call，见[分析四]
```

## 00489737 注册码长度校验与处理

```assembly
**************注册码长为 0x14 跳到这里，前面那一段没有执行

00489737   33DB             xor     ebx, ebx              //计数器清零
00489739   8D85EEFEFFFF     lea     eax, [ebp+$FFFFFEEE]   //eax=0012FABA，设为 String1
的地址
；这个地址和前面初始化$100 个 byte 的首地址相同，
这句执行完后，看到 CPU 下面的提示面板里有：

Stack ss:[0012FBC0]=0101B758, (ASCII "10233219330999945687")  ===>果然不错啊！^\_^
edx=0101B761, (ASCII "30999945687")
Jump from 00489751
Loop：

0048973F   8B55F4           mov     edx, [ebp-$0C]        ;EDX 0101B758 ASCII
"10233219330999945687"
；注册码长为 20，EDX 0101B888 ASCII "12345678901234567890"，这是输入的注册码

00489742   8A141A           mov     dl, byte ptr [edx+ebx]  ；依次取每个字符

00489745   80EA14           sub     dl, $14

00489748   8810             mov     [eax], dl             ；存入 string1 ,byte to
word

  //修正前面初始化的$100 个 byte

0048974A   43               inc     ebx                    ；计数器++

0048974B   83C002           add     eax, +$02              ;Move to the next word

0048974E   83FB14           cmp     ebx, +$14              ;IS the END?

00489751   75EC             jnz     0048973F               ；Goto LOoP

00489753   8BC7             mov     eax, edi               ；eax=00000014，长度   //
取出前面压入的数

00489755   888525FFFFFF     mov     [ebp+$FFFFFF25], al     ；mov byte ptr ss:[ebp-
DB],al //习惯于看减法
                                             ；ebp-DB=0012FAF1

0048975B   8A45F2           mov     al, byte ptr [ebp-$0E]  ；al=05

0048975E   888527FFFFFF     mov     [ebp+$FFFFFF27], al     ；mov [ebp-D9], al
                                             ；ebp-D9=0012FAF3

00489764   8A45F0           mov     al, byte ptr [ebp-$10]  ；al=04

00489767   88852AFFFFFF     mov     [ebp+$FFFFFF2A], al     ；mov byte ptr ss:[ebp-
D6],al
                                             ；ebp-D6=0012FAF6

0048976D   8A45EE           mov     al, byte ptr [ebp-$12]  ；al=1D

00489770   88852DFFFFFF     mov     [ebp+$FFFFFF2D], al     ；mov byte ptr ss:[ebp-
D3],al
                                             ；ebp-D3=0012FAF9

* Possible String Reference to: '2810'
|

00489776   B8D0984800       mov     eax, $004898D0
* Reference to: Unit_004073B8.Proc_0040832C
|

0048977B   E8ACEBF7FF       call    0040832C               ///见后面的[分析三]，
将"2810"转化为十六进制数 0AFA

00489780   8BC8             mov     ecx, eax              //ECX 00000AFA

00489782   8BC1             mov     eax, ecx              //

00489784   BB64000000       mov     ebx, $00000064        //

00489789   99               cdq                            //

0048978A   F7FB             idiv    ebx

；EAX 0000001C
；ECX 00000AFA      ECX=EAX*EBX+EDX
；EDX 0000000A
；EBX 00000064

0048978C   8BD8             mov     ebx, eax              //商

0048978E   8BC3             mov     eax, ebx              //商

00489790   0414             add     al, +$14              //1C+14=30

00489792   88852EFFFFFF     mov     [ebp+$FFFFFF2E], al   ；mov byte ptr ss:[ebp-D2],al
  ，ebp-0d2=0012FAFA，0x100 个 byte 首地址

00489798   2ACB             sub     cl, bl                //FA-1C=DE，将版本信息保存到注
册信息数组中

0048979A   80C114           add     cl, $14               //DE+14=F2
```



## 汇编代码段 1

`0048979D` `888D38FFFFFF` `mov [ebp+$FFFFFF38], cl` ；`mov byte ptr ss:[ebp-C8],cl`

`ebp-C8=0012FB04`

`004897A3` `8D45F8` `lea eax, [ebp-$08]` //`eax=0012FBC4`，不是 `0x100byte` 中的字

* `Reference to: System.Proc_00403B40`

|

`004897A6` `E895A3F7FF` `call 00403B40`

```
{
    00403B40   8B10             mov     edx, [eax]
    00403B42   85D2             test    edx, edx                     //edx=0 就跳了，看来 edx 是个标志
    00403B44   741B             jz      00403B61
    00403B46   C70000000000     mov     dword ptr [eax], $00000000   //不为 0 就置[eax]为 0
    00403B4C   8B4AF8           mov     ecx, [edx-$08]
    00403B4F   49               dec     ecx
    00403B50   7C0F             jl      00403B61
    00403B52   894AF8           mov     [edx-$08], ecx
    00403B55   750A             jnz     00403B61
    00403B57   50               push    eax
    00403B58   8D42F8           lea     eax, [edx-$08]
    * Reference to: system.@FreeMem;
    |
    00403B5B   E8E4EBFFFF       call    00402744
    00403B60   58               pop     eax
    00403B61   C3               ret
}
```

`004897AB` `8D95E0FEFFFF` `lea edx, [ebp+$FFFFFEE0]` ；`lea edx,dword ptr ss:[ebp-120]` `edx=0012FAAC`

* `Reference to AboutBox`

|

`004897B1` `A154494D00` `mov eax, dword ptr [$004D4954]` ；`eax=00FF798C`

* `Reference to control edit_name : N.A.` `*****************获得用户名*****************`

|

`004897B6` `8B80E4020000` `mov eax, [eax+$02E4]` ；`eax=00FDDA84`

* `Reference to: controls.TControl.GetText(TControl):TCaption;`

|

`004897BC` `E84F16FAFF` `call 0042AE10`

`004897C1` `8B85E0FEFFFF` `mov eax, [ebp+$FFFFFEE0]` ；`EAX 0101B710 ASCII "CNwinndy"`

`004897C7` `8D55F4` `lea edx, [ebp-$0C]` ；`EDX 0012FBC0`

* `Reference to: Unit_004073B8.Proc_00408148`

|

`004897CA` `E879E9F7FF` `call 00408148` ；第一次跟进了没发现什么，第二次 `F8` 跳过

`004897CF` `8B45F4` `mov eax, [ebp-$0C]` ；`EAX 0101B77C ASCII "CNwinndy"`

* `Reference to: System.Proc_00403DBC`

|

`004897D2` `E8E5A5F7FF` `call 00403DBC`

```
{
    00403DBC   85C0             test    eax, eax
    00403DBE   7403             jz      00403DC3
    00403DC0   8B40FC           mov     eax, [eax-$04]
    00403DC3   C3               ret
}
```

`004897D7` `8BD0` `mov edx, eax` ；`eax=00000008`，用户名长度

`004897D9` `85D2` `test edx, edx`

`004897DB` `7E1C` `jle 004897F9` ；用户名为空就跳

`004897DD` `BB01000000` `mov ebx, $00000001` ；计数器初值

`004897E2` `8D8559FFFFFF` `lea eax, [ebp+$FFFFFF59]` ；`lea eax,dword ptr ss:[ebp-A7]` `eax=0012FB25`，设为 `String2` 的首地址

`004897E8` `8B4DF4` `mov ecx, [ebp-$0C]` ；`ECX 0101B77C ASCII "CNwinndy"`

`loop：`

`004897EB` `8A4C19FF` `mov cl, byte ptr [ecx+ebx-$01]`

`004897EF` `80E90F` `sub cl, $0F`

`004897F2` `8808` `mov [eax], cl` ；保存到 `String2`

`004897F4` `43` `inc ebx`

`004897F5` `40` `inc eax`

`004897F6` `4A` `dec edx`

`004897F7` `75EF` `jnz 004897E8` ；`goto loop`

；`0012FB25 34 3F 68 5A 5F 5F 55 6A 4?hZ__Uj =====>用户名`

`004897F9` `C6841D58FFFFFF00` `mov byte ptr [ebp+ebx+$FFFFFF58], $00` ；`mov byte ptr ss:[ebp+ebx-A8],0` 置字符串结尾标志`\\0`

`00489801` `6800010000` `push $00000100`

`00489806` `8D8DEEFEFFFF` `lea ecx, [ebp+$FFFFFEEE]` ；`lea ecx,dword ptr ss:[ebp-112]` `ecx=0012FABA`

* `Possible String Reference to: 'WCID'`

|

`0048980C` `BAC0984800` `mov edx, $004898C0`

`00489811` `8B45FC` `mov eax, [ebp-$04]`

* `Reference to: Unit_0044EDF8.Proc_0044F884`

|

`00489814` `E86B60FCFF` `call 0044F884` ； `F8` 跳过

```
{0044F884   55               push    ebp
    0044F885   8BEC             mov     ebp, esp
    0044F887   53               push    ebx
    0044F888   8B5D08           mov     ebx, [ebp+$08]
    0044F88B   53               push    ebx
    0044F88C   6A04             push    $04
    * Reference to: Unit_0044EDF8.Proc_0044F8FC
    |
    0044F88E   E869000000       call    0044F8FC       //有关注册表操作，进去一遍后没发现有价值的，下次可跳过
    0044F893   5B               pop     ebx
    0044F894   5D               pop     ebp
    0044F895   C20400           ret     $0004
}
```

`00489819` `8B45FC` `mov eax, [ebp-$04]` //`eax=0101B6EC`

* `Reference to: Unit_0044EDF8.Proc_0044F000`

|

`0048981C` `E8DF57FCFF` `call 0044F000` //有关注册表的操作，进去一遍后没发现有价值的，下次可跳过

`00489821` `8B45FC` `mov eax, [ebp-$04]` //`eax=0101B6EC`

* `Reference to: system.TObject.Free(TObject);`

|

`00489824` `E84797F7FF` `call 00402F70` //`F8` 跳过

`00489829` `33C0` `xor eax, eax`

`0048982B` `5A` `pop edx`

`0048982C` `59` `pop ecx`

`0048982D` `59` `pop ecx`

`0048982E` `648910` `mov fs:[eax], edx`

`****** FINALLY`

|

* `Possible String Reference to: '_^[\u5995\u5A25]?`

|

`00489831` `6866984800` `push $00489866`

`00489836` `8D85E0FEFFFF` `lea eax, [ebp+$FFFFFEE0]` ；`lea eax,dword ptr ss:[ebp-120]` //`eax=0012FAAC`

； `D [eax]`

； `0101B710 43 4E 77 69 6E 6E 64 79 CNwinndy`

； `0101B718 00 .`

* `Reference to: System.Proc_00403B40`

|

`0048983C` `E8FFA2F7FF` `call 00403B40` ；`F8`

`00489841` `8D85E4FEFFFF` `lea eax, [ebp+$FFFFFEE4]` ；`lea eax,dword ptr ss:[ebp-11C]` //`eax=0012FAB0`

；`D [eax]`

；`0101B740 31 30 32 33 33 32 31 39 10233219`

；`0101B748 33 00 3.`

`00489847` `BA02000000` `mov edx, $00000002`

* `Reference to: System.Proc_00403B64`

|

`0048984C` `E813A3F7FF` `call 00403B64` ；`F8`

`00489851` `8D45F4` `lea eax, [ebp-$0C]`

`00489854` `BA02000000` `mov edx, $00000002`

* `Reference to: System.Proc_00403B64`

|

`00489859` `E806A3F7FF` `call 00403B64` ；`F8`

`0048985E` `C3` `ret` ；回到 `00489866`

* `Reference to: System.Proc_004035E4`

|

`0048985F` `E9809DF7FF` `jmp 004035E4`

`00489864` `EBD0` `jmp 00489836`

`****** END`

|

`00489866` `5F` `pop edi`

`00489867` `5E` `pop esi`

`00489868` `5B` `pop ebx`

`00489869` `8BE5` `mov esp, ebp`

`0048986B` `5D` `pop ebp`

`0048986C` `C3` `ret` ；回到 `004894CD`，见上面

===================================================================================

## [分析二] `******` 很重要，也很长

`004AD8D0` `55` `push ebp`

`004AD8D1` `8BEC` `mov ebp, esp`

`004AD8D3` `81C4E4FEFFFF` `add esp, $FFFFFEE4`

`004AD8D9` `53` `push ebx`



```
004AD8DA   56                push    esi  
004AD8DB   57                push    edi  
004AD8DC   33C0             xor     eax, eax  
004AD8DE   8985E8FEFFFF     mov     [ebp+$FFFFFEE8], eax；ebp-118=0012FAB4  
004AD8E4   8945F8           mov     [ebp-$08], eax  
004AD8E7   8945F4           mov     [ebp-$0C], eax  
004AD8EA   8945F0           mov     [ebp-$10], eax  
004AD8ED   8945EC           mov     [ebp-$14], eax      ；ebp-14=0012FBB8  
004AD8F0   33C0             xor     eax, eax  
004AD8F2   55               push    ebp  
* Possible String Reference to: '轫 Y?豚嬅_^[嬪]?

004AD8F3   68F2DB4A00       push    $004ADBF2  
***** TRY

004AD8F8   64FF30           push    dword ptr fs:[eax]  
004AD8FB   648920           mov     fs:[eax], esp  
004AD8FE   B201             mov     dl, $01  
* Reference to class TRegistry

004AD900   A158EE4400       mov     eax, dword ptr [$0044EE58]  
* Reference to: Unit_0044EDF8.Proc_0044EF98

004AD905   E88E16FAFF       call    0044EF98           ；F8 跳过  
004AD90A   8BF8             mov     edi, eax  
004AD90C   BA01000080       mov     edx, $80000001  
004AD911   8BC7             mov     eax, edi  
* Reference to: Unit_0044EDF8.Proc_0044F030

004AD913   E81817FAFF       call    0044F030           ；F8 跳过  
004AD918   B101             mov     cl, $01  
* Possible String Reference to: '\Software\Microsoft\Windows\Current

        Version\Explorer\Advanced'

004AD91A   BA0CDC4A00       mov     edx, $004ADC0C  
004AD91F   8BC7             mov     eax, edi  
* Reference to: Unit_0044EDF8.Proc_0044F18C

004AD921   E86618FAFF       call    0044F18C           ；F8 跳过  
* Possible String Reference to: 'WCID'

004AD926   BA54DC4A00       mov     edx, $004ADC54  
004AD92B   8BC7             mov     eax, edi  
* Reference to: Unit_0044EDF8.Proc_0044FA50

004AD92D   E81E21FAFF       call    0044FA50       ；进去看  
   {0044FA50  /$  83C4 F8         add esp,-8  
    0044FA53  |.  8BCC            mov ecx,esp  
    0044FA55  |.  E8 C6FCFFFF      call talisman.0044F720 //*  
advapi32.RegQueryValueExA()  
    0044FA5A  |.  59              pop ecx  
    0044FA5B  |.  5A              pop edx  
    0044FA5C  \.  C3              retn  
   }  
004AD932   84C0             test    al, al  
004AD934   0F85B2000000     jnz     004AD9EC       ；跳了，跳过去的这一段就不看  
了  
004AD93A   BB33000000       mov     ebx, $00000033  
004AD93F   8DB5ECFEFFFF     lea     esi, [ebp+$FFFFFEEC]  
004AD945   B809000000       mov     eax, $00000009  
* Reference to: system.@RandInt;

004AD94A   E83D53F5FF       call    00402C8C  
004AD94F   041C             add     al, +$1C  
004AD951   8806             mov     [esi], al  
004AD953   46               inc     esi  
004AD954   4B               dec     ebx  
004AD955   75EE             jnz     004AD945  
004AD957   BBCD000000       mov     ebx, $000000CD  
004AD95C   8DB51FFFFFFF     lea     esi, [ebp+$FFFFFF1F]  
004AD962   B8FF000000       mov     eax, $000000FF  
* Reference to: system.@RandInt;

004AD967   E82053F5FF       call    00402C8C  
004AD96C   8806             mov     [esi], al  
004AD96E   46               inc     esi  
004AD96F   4B               dec     ebx  
004AD970   75F0             jnz     004AD962  
004AD972   C68557FFFFFF00   mov     byte ptr [ebp+$FFFFFF57], $00  
004AD979   C68523FFFFFF01   mov     byte ptr [ebp+$FFFFFF23], $01  
004AD980   C68525FFFFFF01   mov     byte ptr [ebp+$FFFFFF25], $01  
004AD987   C68528FFFFFF01   mov     byte ptr [ebp+$FFFFFF28], $01  
004AD98E   C6852BFFFFFF01   mov     byte ptr [ebp+$FFFFFF2B], $01  
* Possible String Reference to: '2810'

004AD995   B864DC4A00       mov     eax, $004ADC64  
* Reference to: Unit_004073B8.Proc_0040832C

004AD99A   E88DA9F5FF       call    0040832C  
004AD99F   8BD8             mov     ebx, eax  
004AD9A1   8BC3             mov     eax, ebx  
004AD9A3   B964000000       mov     ecx, $00000064  
004AD9A8   99               cdq  
004AD9A9   F7F9             idiv    ecx  
004AD9AB   8BC8             mov     ecx, eax  
004AD9AD   8BC1             mov     eax, ecx  
004AD9AF   0414             add     al, +$14  
004AD9B1   88852CFFFFFF     mov     [ebp+$FFFFFF2C], al  
004AD9B7   8BC1             mov     eax, ecx  
004AD9B9   6BC064           imul    eax, eax, $64  
004AD9BC   2AD8             sub     bl, al  
004AD9BE   80C314           add     bl, $14  
004AD9C1   889D36FFFFFF     mov     [ebp+$FFFFFF36], bl  
004AD9C7   6800010000       push    $00000100  
004AD9CC   8D8DECFEFFFF     lea     ecx, [ebp+$FFFFFEEC]  
* Possible String Reference to: 'WCID'

004AD9D2   BA54DC4A00       mov     edx, $004ADC54  
004AD9D7   8BC7             mov     eax, edi  
* Reference to: Unit_0044EDF8.Proc_0044F884

004AD9D9   E8A61EFAFF       call    0044F884  
* Reference to TForm1 instance

004AD9DE   A11C364C00       mov     eax, dword ptr [$004C361C]  
004AD9E3   8B00             mov     eax, [eax]  
* Reference to field TForm1.Tag : Longint

004AD9E5   C7400C01000000   mov     dword ptr [eax+$0C], $00000001  
                                ***跳到这里***  
004AD9EC   6800010000       push    $00000100  
004AD9F1   8D8DECFEFFFF     lea     ecx, [ebp+$FFFFFEEC]  
* Possible String Reference to: 'WCID'

004AD9F7   BA54DC4A00       mov     edx, $004ADC54  
004AD9FC   8BC7             mov     eax, edi  
* Reference to: Unit_0044EDF8.Proc_0044F898

004AD9FE   E8951EFAFF       call    0044F898    ；F8 跳过  
004ADA03   8D45F0           lea     eax, [ebp-$10]  
* Reference to: System.Proc_00403B40

004ADA06   E83561F5FF       call    00403B40    ；F8 跳过  
004ADA0B   BB14000000       mov     ebx, $00000014        ；常数  
串"10233219330999945687"的长度  
004ADA10   8DB5ECFEFFFF     lea     esi, [ebp+$FFFFFEEC]     ；lea esi,dword ptr ss:  
[ebp-114]；esi=0012FAB8  
  ；D [ESI]                                  
 ；0012FAB8  1D 39 1C 91 1E D7 1F 7B  9??{            "10233219330999945687"经过处理得出  
的数据  
；0012FAC0  1F C0 1E 90 1D 3A 25 6D  ??:%m            从第一个开始，每次步长为 2，隔一个  
；0012FAC8  1F 0C 1F 9F 1C FB 25 FA  .??  
；0012FAD0  25 97 25 93 25 05 20 BA  %??  
；0012FAD8  21 DD 22 91 24 49 23 1A  !??I#  
Loop：  
004ADA16   8D85E8FEFFFF     lea     eax, [ebp+$FFFFFEE8]     ；lea eax,dword ptr ss:  
[ebp-118]，eax=0012FAB4  
004ADA1C   33D2             xor     edx, edx  
004ADA1E   8A16             mov     dl, byte ptr [esi]  
004ADA20   83C214           add     edx, +$14        ；  
* Reference to: System.Proc_00403CE4

004ADA23   E8BC62F5FF       call    00403CE4     //0012FAB4 中的是 system.Move 返回的  
地址，每次都会变化  
   { 00403CE4  /$  52              push edx  
     00403CE5  |.  89E2            mov edx,esp  
     00403CE7  |.  B9 01000000     mov ecx,1  
     00403CEC  |.  E8 33FFFFFF     call talisman.00403C24          ；  
*system.Move(void;void;void;void;Integer);  
     00403CF1  |.  5A              pop edx  
     00403CF2  \.  C3              retn  
   }  
```



# 汇编代码分析

## 循环与字符串提取
```
004ADA28   8B95E8FEFFFF           mov     edx, [ebp+$FFFFFEE8]              ；  

edx=00FDDD08  

004ADA2E   8D45F0                 lea     eax, [ebp-$10]                    ；  

eax=0012FBBC  

* Reference to: `System.Proc_00403DC4`  

|  

004ADA31   E88E63F5FF             call    00403DC4                          ；* 

`system.Move(void;void;void;void;Integer);`

004ADA36   83C602                 add     esi, +$02  

004ADA39   4B                     dec     ebx  

004ADA3A   75DA                   jnz     004ADA16               ；goto Loop  

004ADA3C   8D45EC                 lea     eax, [ebp-$14]         ；eax=0012FBB8  

；d eax  

；0012FBB8  00 00 00 00 10 B7 01 01  ....?  

；d 0101B710  

；0101B710  31 30 32 33 33 32 31 39  10233219  

；0101B718  33 33 30 39 39 39 39 34  33099994  

；0101B720  35 36 38 37 00             5687.  

004ADA3F   50                     push    eax              ；0012FBB8  

004ADA40   B902000000             mov     ecx, $00000002  

004ADA45   BA0E000000             mov     edx, $0000000E      //MID(eax,edx-1,ecx)，特别注意  

004ADA4A   8B45F0                 mov     eax, [ebp-$10]      //EAX 0101B710 ASCII 

"10233219330999945687"  

* Reference to: `System.Proc_00403FC0`  

|  

004ADA4D   E86E65F5FF             call    00403FC0          //先 F8，后来发现这步生成的结果后

面用上了，进去看看  

      {  
        00403FC0    /$  53             push ebx  

        00403FC1    |.  85C0           test eax,eax  

        00403FC3    |.  74 2D          je short `talisman.00403FF2`  

        00403FC5    |.  8B58 FC        mov ebx,dword ptr ds:[eax-4]      

//ebx=14，"10233219330999945687"的长度  

        00403FC8    |.  85DB           test ebx,ebx  

        00403FCA    |.  74 26          je short `talisman.00403FF2`  

        00403FCC    |.  4A             dec edx                      

//edx=0000000E==>0000000D  

        00403FCD    |.  7C 1B          jl short `talisman.00403FEA`  

        00403FCF    |.  39DA           cmp edx,ebx                   

        00403FD1    |.  7D 1F          jge short `talisman.00403FF2`  

        00403FD3    |>  29D3           sub ebx,edx                //ebx=14-0D=07  

        00403FD5    |.  85C9           test ecx,ecx               //ecx=02  

        00403FD7    |.  7C 19          jl short `talisman.00403FF2`  

        00403FD9    |.  39D9           cmp ecx,ebx                 

        00403FDB    |.  7F 11          jg short `talisman.00403FEE`  

        00403FDD    |>  01C2           add edx,eax                //EDX 0101B87D  

ASCII "9945687"  

        00403FDF    |.  8B4424 08      mov eax,dword ptr ss:[esp+8]     //0012FBB8  

        00403FE3    |.  E8 3CFCFFFF    call `talisman.00403C24`        //F8  

        00403FE8    |.  EB 11          jmp short `talisman.00403FFB`  

        00403FEA    |>  31D2           xor edx,edx  

        00403FEC    |.^ EB E5          jmp short `talisman.00403FD3`  

        00403FEE    |>  89D9           mov ecx,ebx  

        00403FF0    |.^ EB EB          jmp short `talisman.00403FDD`  

        00403FF2    |>  8B4424 08      mov eax,dword ptr ss:[esp+8]  

        00403FF6    |.  E8 45FBFFFF    call `talisman.00403B40`  

        00403FFB    |>  5B             pop ebx  

        00403FFC    \\.  C2 0400        retn 4  

      }  

004ADA52   8B45EC                 mov     eax, [ebp-$14]     //EAX 00FDDD08 ASCII "99"，  

* Reference to: `Unit_004073B8.Proc_0040832C`  

|  

004ADA55   E8D2A8F5FF             call    0040832C          //前面分析过，[分析三]，版本

号"2810"变为十六进制  

004ADA5A   8945FC                 mov     [ebp-$04], eax    //eax=00000063，这是 99 的十六进制  

004ADA5D   8D45EC                 lea     eax, [ebp-$14]     D [eax] 39 39 00          "99"  

004ADA60   50                     push    eax  

004ADA61   B908000000             mov     ecx, $00000008     //特别注意这个 MID 函数  

004ADA66   BA06000000             mov     edx, $00000006    //MID(eax,edx-1,ecx),21933099,hex 

14EAC2B  

004ADA6B   8B45F0                 mov     eax, [ebp-$10]    //  

；  EAX 0101B710 ASCII "10233219330999945687"  

；  ECX 00000008  

；  EDX 00000006  

；  EBX 00000000  

；  ESP 0012FA94  

；  EBP 0012FBCC  

；  ESI 0012FAE0  

；  EDI 0101B6EC ASCII "ゎD"  

* Reference to: `System.Proc_00403FC0`  

|  

004ADA6E   E84D65F5FF             call    00403FC0                    ；F8,MID string function  

004ADA73   8D45F8                 lea     eax, [ebp-$08]           ；0012FBC4  

* Reference to: `System.Proc_00403B40`  

|  

004ADA76   E8C560F5FF             call    00403B40                   ;F8  

004ADA7B   BB95000000             mov     ebx, $00000095           //常数  

004ADA80   8DB557FFFFFF           lea     esi, [ebp+$FFFFFF57]     //ESI 0012FB23 ASCII "4?

hZ_Uj"  这是用户名处理后的结果，参见 004897F7  

    Loop：  

004ADA86   803E00                 cmp     byte ptr [esi], $00  

004ADA89   7424                   jz      004ADAAF  

004ADA8B   8D85E8FEFFFF           lea     eax, [ebp+$FFFFFEE8]     //eax=0012FAB4  

004ADA91   33D2                   xor     edx, edx  

004ADA93   8A16                   mov     dl, byte ptr [esi]  

004ADA95   83C20F                 add     edx, +$0F              //还原为原来的字符  

* Reference to: `System.Proc_00403CE4`  

|  

004ADA98   E84762F5FF             call    00403CE4                 //F8  

004ADA9D   8B95E8FEFFFF           mov     edx, [ebp+$FFFFFEE8]     //EDX 01004514  

004ADAA3   8D45F8                 lea     eax, [ebp-$08]           //EAX 0012FBC4  

* Reference to: `System.Proc_00403DC4`  

|  

004ADAA6   E81963F5FF             call    00403DC4     

004ADAAB   46                     inc     esi  

004ADAAC   4B                     dec     ebx  

004ADAAD   75D7                   jnz     004ADA86     ；goto Loop  

004ADAAF   33F6                   xor     esi, esi  

004ADAB1   8B45F8                 mov     eax, [ebp-$08]  ；EAX 0101B74C ASCII "CNwinndy"  

* Reference to: `System.Proc_00403DBC`  

|  

004ADAB4   E80363F5FF             call    00403DBC          //  

{  

     00403DBC   /$  85C0           test eax,eax           ；用户名是否为空  

     00403DBE   |.  74 03          je short `talisman.00403DC3`  

     00403DC0   |.  8B40 FC        mov eax,dword ptr ds:[eax-4]  ；EAX=00000008，用户名长

度  

     00403DC3   \\.>  C3             retn  

  }  

004ADAB9   85C0                   test    eax, eax  

004ADABB   7E13                   jle     004ADAD0  

004ADABD   BB01000000             mov     ebx, $00000001           //计数器初值  

LOOP：                                                   //这段循环计算用户名的 ascii

累加和  

004ADAC2   8B55F8                 mov     edx, [ebp-$08]             //EDX 0101B74C ASCII 

"CNwinndy"  

004ADAC5   0FB6541AFF             movzx   edx, byte ptr [edx+ebx-$01]  

004ADACA   03F2                   add     esi, edx               //用户名的累加和，最终为

0000032A  

004ADACC   43                     inc     ebx  

004ADACD   48                     dec     eax  

004ADACE   75F2                   jnz     004ADAC2           //goto loop  

004ADAD0   8D55F4                 lea     edx, [ebp-$0C]        //edx=0012FBC0  

004ADAD3   8B45FC                 mov     eax, [ebp-$04]        //eax=00000063,"99"变来

；D ebp-0c  

；0012FBC0  00 00 00 00 4C B7 01 01  ....L?  

；0012FBC8  63 00 00 00               c...  

；d 0101B74C  

；0101B74C  43 4E 77 69 6E 6E 64 79  CNwinndy  

；0101B754  00                       .  

* Reference to: `Unit_004AC9A4.Proc_004AD1D0`  

|  

004ADAD6   E8F5F6FFFF             call    004AD1D0               //******暂时 F8 略过，第二遍需

进去，因为不知道下面的串是怎么生成的，见下面[分析五]  

                                                   //case 1-15，55-69  

004ADADB   8B45F4                 mov     eax, [ebp-$0C]         //EAX 0101B77C ASCII "99 

99632671"
```



## Reference to: `System.Proc_00403DBC`

```
004ADADE E8D962F5FF         call 00403DBC             //暂时 F8 掠过
004ADAE3 83F808             cmp eax, +$08             //EAX 0000000B，应该是"99 99632671"的长度
004ADAE6 7D12              jnl 004ADAFA              ；跳了
004ADAE8 8D4DF4            lea ecx, [ebp-$0C]
```

## Reference to `TForm1` instance

```
004ADAEB A11C364C00        mov eax, dword ptr [$004C361C]
004ADAF0 8B00              mov eax, [eax]
004ADAF2 8B55FC            mov edx, [ebp-$04]
```

## Reference to: `t2_main.Proc_004B4260`

```
004ADAF5 E866670000        call 004B4260             //**********见下面 [分析六]   
//case 16-2A
004ADAFA 8B45F4            mov eax, [ebp-$0C]        //EAX 0101B77C ASCII "99 99632671"
```

## Reference to: `System.Proc_00403DBC`

```
004ADAFD E8BA62F5FF        call 00403DBC
004ADB02 83F808            cmp eax, +$08
004ADB05 7D12             jnl 004ADB19              //跳了
004ADB07 8D4DF4           lea ecx, [ebp-$0C]
```

## Reference to `TForm1` instance

```
004ADB0A A11C364C00       mov eax, dword ptr [$004C361C]
004ADB0F 8B00             mov eax, [eax]
004ADB11 8B55FC           mov edx, [ebp-$04]
```

## Reference to: `t2_main.Proc_004BBCE0`

```
004ADB14 E8C7E10000       call 004BBCE0              //************见下面[分析七]//case 2B-3F
004ADB19 8B45F4           mov eax, [ebp-$0C]         //EAX 0101B77C ASCII "99 99632671"
```

## Reference to: `System.Proc_00403DBC`

```
004ADB1C E89B62F5FF       call 00403DBC
004ADB21 83F808           cmp eax, +$08
004ADB24 7D0B            jnl 004ADB31               //跳了
004ADB26 8D55F4          lea edx, [ebp-$0C]
004ADB29 8B45FC          mov eax, [ebp-$04]
```

## Reference to: `Unit_004AC9A4.Proc_004ACECC`

```
004ADB2C E89BF3FFFF       call 004ACECC             //******见下面[分析八]  //case 40-54
004ADB31 8B55F4           mov edx, [ebp-$0C]       //EDX 0101B77C ASCII "99 99632671"
004ADB34 B874DC4A00       mov eax, $004ADC74       //EAX 004ADC74 talisman.004ADC74
```

## Reference to: `System.Proc_004040A4`

```
004ADB39 E86665F5FF       call 004040A4             //F8
004ADB3E 8BD8             mov ebx, eax             //eax=0003
004ADB40 8D45F4           lea eax, [ebp-$0C]
；D eax
；0101B77C  39 39 20 39 39 36 33 32  99 99632
；0101B784  36 37 31 00              671.
004ADB43 8BCB            mov ecx, ebx
004ADB45 BA01000000      mov edx, $00000001
```

## Reference to: `System.Proc_00404000`

```
004ADB4A E8B164F5FF       call 00404000             //F8
004ADB4F 8B45F4           mov eax, [ebp-$0C]         //EAX 0101B77C ASCII "99632671"
```

## Reference to: `Unit_004073B8.Proc_0040832C`

```
004ADB52 E8D5A7F5FF       call 0040832C             //前面分析过，数字串，转化为十六进制
004ADB57 8BD8             mov ebx, eax             //EAX 05F0461F "99632671"的十六进制
004ADB59 89B5E4FEFFFF     mov [ebp+$FFFFFEE4], esi  //ESI 0000032A ，用户名 的累加和
004ADB5F DB85E4FEFFFF     fild dword ptr [ebp+$FFFFFEE4]
```

## Reference to: `system.@ROUND;`

```
004ADB65 E84E4FF5FF       call 00402AB8             //EAX 0000032A
004ADB6A 69C067030000     imul eax, eax, $00000367   //eax=000AC3E6 =32A*367   
   这里很重要
004ADB70 03D8             add ebx, eax             
//ebx=05F0461F+000AC3E6=05FB0A05
004ADB72 8BF3             mov esi, ebx
004ADB74 8D95E8FEFFFF     lea edx, [ebp+$FFFFFEE8]   //edx=0012FAB4
004ADB7A 8BC6             mov eax, esi
```

## Reference to: `Unit_004073B8.Proc_004082C8`

```
004ADB7C E847A7F5FF       call 004082C8             //F8 跳过，看结果
；EAX 0101B734 ASCII "21933099"=======>14EAC2B 的十进制  
；ECX 05FB0A05
；EDX 0101B764 ASCII "100338181" ====05FB0A05 的十进制
；EBX 05FB0A05
004ADB81 8B95E8FEFFFF     mov edx, [ebp+$FFFFFEE8]    
004ADB87 8B45EC            mov eax, [ebp-$14]
```

## Reference to: `System.Proc_00403ECC`

```
004ADB8A E83D63F5FF       call 00403ECC             //F8 跳过后，看到 EAX=FFFFFFFF
，这里要跳了 见[分析九]，功能是比较字符串
；看来这个 call 是比较注册码了，"100338181"与"21933099"是否相同。
；观察这个跳后的程序与不跳的程序，可以发现一点，跳了之后 Xor ebx，ebx，即 ebx 清零；而不跳则 mov 
bl，1，置 1；
；这应该是标志了。不妨双击'Z',改变流程看如何。
004ADB8F 751D             jnz 004ADBAE
```

## Reference to pointer to `GlobalVar_004D4AA8`

```
004ADB91 A1A8314C00       mov eax, dword ptr [$004C31A8]
004ADB96 8B55F8            mov edx, [ebp-$08]        //EDX 0101B74C ASCII "CNwinndy"
```

## Reference to: `System.Proc_00403B94`

```
004ADB99 E8F65FF5FF       call 00403B94             //F8
```

## Reference to `TForm1` instance

```
004ADB9E A11C364C00       mov eax, dword ptr [$004C361C]
004ADBA3 8B00             mov eax, [eax]
004ADBA5 33D2             xor edx, edx
```

## Reference to field `TForm1.Tag` : `Longint`

```
004ADBA7 89500C            mov [eax+$0C], edx
004ADBAA B301              mov bl, $01
004ADBAC EB10              jmp 004ADBBE
```

## Reference to `TForm1` instance

```
004ADBAE A11C364C00       mov eax, dword ptr [$004C361C]
004ADBB3 8B00             mov eax, [eax]
```

## Reference to field `TForm1.Tag` : `Longint`

```
004ADBB5 C7400C01000000    mov dword ptr [eax+$0C], $00000001
004ADBBC 33DB              xor ebx, ebx
004ADBBE 8BC7              mov eax, edi
```

## Reference to: `Unit_0044EDF8.Proc_0044F000`

```
004ADBC0 E83B14FAFF        call 0044F000
004ADBC5 8BC7              mov eax, edi
```

## Reference to: `system.TObject.Free(TObject);`

```
004ADBC7 E8A453F5FF        call 00402F70
004ADBCC 33C0               xor eax, eax
004ADBCE 5A                pop edx
004ADBCF 59                pop ecx
004ADBD0 59                pop ecx
004ADBD1 648910             mov fs:[eax], edx
```

****** FINALLY

## Possible String Reference to: `'嬅_^\[嬪\]?'`

```
004ADBD4 68F9DB4A00         push $004ADBF9
004ADBD9 8D85E8FEFFFF       lea eax, [ebp+$FFFFFEE8]
```

## Reference to: `System.Proc_00403B40`

```
004ADBDF E85C5FF5FF        call 00403B40
004ADBE4 8D45EC            lea eax, [ebp-$14]
004ADBE7 BA04000000        mov edx, $00000004
```

## Reference to: `System.Proc_00403B64`

```
004ADBEC E8735FF5FF        call 00403B64
004ADBF1 C3                ret
```

## Reference to: `System.Proc_004035E4`

```
004ADBF2 E9ED59F5FF        jmp 004035E4
004ADBF7 EBE0              jmp 004ADBD9
```

****** END

```
004ADBF9 8BC3              mov eax, ebx
004ADBFB 5F                pop edi
004ADBFC 5E                pop esi
004ADBFD 5B                pop ebx
004ADBFE 8BE5              mov esp, ebp
004ADC00 5D                pop ebp
004ADC01 C3                ret                ；返回到 004894D2，见上面
```

================================================================

### 分析三

```
0040832C 55                push ebp
0040832D 8BEC              mov ebp, esp
0040832F 83C4F0             add esp, -$10
```



### 00408332

```asm
00408332   53                            push    ebx
00408333   56                            push    esi
00408334   33D2                          xor     edx, edx
00408336   8955F8                        mov     [ebp-8], edx
00408339   8BD8                          mov     ebx, eax              ; EBX 004898D0 ASCII
                                          "2810"
0040833B   33C0                          xor     eax, eax
0040833D   55                            push    ebp
```

* Possible String Reference to: `'镵?腽嬈^\[嬪\]脨 SQ 嬟嬙桴??$'`

```asm
0040833E   6894834000                    push    $00408394

******** TRY

00408343   64FF30                        push    dword ptr fs:[eax]
00408346   648920                        mov     fs:[eax], esp
00408349   8D55FC                        lea     edx, [ebp-4]
0040834C   8BC3                          mov     eax, ebx
```

* Reference to: `system.@ValLong;`

```asm
0040834E   E851A9FFFF                    call    00402CA4            //F8 过去，根据结果再猜测
                                          其功能
                                          ; eax=00000AFA，正是 2810 的十六进制，马上想到这是什么意思了，呵呵，这个版本是 Bulid 2810。

00408353   8BF0                          mov     esi, eax
00408355   837DFC00                      cmp     dword ptr [ebp-4], +$00
00408359   7423                          jz      0040837E            //在这里跳了
0040835B   8D55F8                        lea     edx, [ebp-8]
0040835E   A1C0324C00                    mov     eax, dword ptr [$004C32C0]
```

* Reference to: `System.Proc_00405584`

```asm
00408363   E81CD2FFFF                    call    00405584
00408368   8B45F8                        mov     eax, [ebp-8]
0040836B   50                            push    eax
0040836C   895DF0                        mov     [ebp-16], ebx
0040836F   C645F40B                      mov     byte ptr [ebp-12], $0B
00408373   8D55F0                        lea     edx, [ebp-16]
00408376   33C9                          xor     ecx, ecx
00408378   58                            pop     eax
```

* Reference to: `Unit_004073B8.Proc_00407EBC`

```asm
00408379   E83EFBFFFF                    call    00407EBC
0040837E   33C0                          xor     eax, eax
00408380   5A                            pop     edx
00408381   59                            pop     ecx
00408382   59                            pop     ecx
00408383   648910                        mov     fs:[eax], edx

******* FINALLY

00408386   689B834000                    push    $0040839B
0040838B   8D45F8                        lea     eax, [ebp-8]
```

* Reference to: `System.Proc_00403B40`

```asm
0040838E   E8ADB7FFFF                    call    00403B40
00408393   C3                            ret     //返回到 0040839B，就在下面
```

* Reference to: `System.Proc_004035E4`

```asm
00408394   E94BB2FFFF                    jmp     004035E4
00408399   EBF0                          jmp     0040838B

****** END

0040839B   8BC6                          mov     eax, esi
0040839D   5E                            pop     esi
0040839E   5B                            pop     ebx
0040839F   8BE5                          mov     esp, ebp
004083A1   5D                            pop     ebp
```

；这时候的寄存器状态为：
；`EAX 00000AFA`              这是"2810"
；`ECX 0012FA8C`
；`EDX 00000000`
；`EBX 00000014`
；`ESP 0012FA90 ASCII "€桯"`
；`EBP 0012FBCC`
；`ESI 0012FBBA`
；`EDI 00000014`

```asm
004083A2   C3                            ret   ;回到 00489780   见上面
```

==================================================================

### [分析四]

```asm
00403DC4    $   85D2                      test edx,edx                   ;EDX 0101B728 ASCII
                                                   "30999945687"
00403DC6    .   74 3F                     je short talisman.00403E07
00403DC8    .   8B08                      mov ecx,dword ptr ds:[eax]     ;ECX 0101B740 ASCII
                                                   "102332193"
00403DCA    .   85C9                      test ecx,ecx
00403DCC    .^  0F84 C2FDFFFF             je talisman.00403B94
00403DD2    .   53                        push ebx
00403DD3    .   56                        push esi
00403DD4    .   57                        push edi
00403DD5    .   89C3                      mov ebx,eax
00403DD7    .   89D6                      mov esi,edx
00403DD9    .   8B79 FC                   mov edi,dword ptr ds:[ecx-4]   ;edi=09，ASCII "102332193"
                                                   的长度
00403DDC    .   8B56 FC                   mov edx,dword ptr ds:[esi-4]   ;edx=0B，ASCII
                                                   "30999945687"的长度
00403DDF    .   01FA                      add edx,edi                     ;长度 1+长度 2，难道要连接这
                                                   两个字符串？
00403DE1    .   39CE                      cmp esi,ecx
00403DE3    .   74 17                     je short talisman.00403DFC
00403DE5    .   E8 02030000               call talisman.004040EC          ;F8 跳过
00403DEA    .   89F0                      mov eax,esi                   ;EAX 0101B728 ASCII
                                                   "30999945687"
00403DEC    .   8B4E FC                   mov ecx,dword ptr ds:[esi-4]   ;ECX 0000000B
00403DEF    >   8B13                      mov edx,dword ptr ds:[ebx]     ;EDX 0101B758 ASCII
                                                   "102332193"
00403DF1    .   01FA                      add edx,edi                   ;EDI 00000009
00403DF3    .   E8 C0EAFFFF               call talisman.004028B8        ;里面有 system.Move
                                                   (void;void;void;void;Integer)
00403DF8    .   5F                        pop edi
00403DF9    .   5E                        pop esi
00403DFA    .   5B                        pop ebx
00403DFB    .   C3                        retn                           ;回到 00489737
00403DFC    >   E8 EB020000               call talisman.004040EC
00403E01    .   8B03                      mov eax,dword ptr ds:[ebx]
00403E03    .   89F9                      mov ecx,edi
00403E05    .^  EB E8                     jmp short talisman.00403DEF
00403E07    >   C3                        retn
```

```asm
00403E08   85D2                          test    edx, edx
00403E0A   7461                          jz      00403E6D
00403E0C   85C9                          test    ecx, ecx
00403E0E   0F8480FDFFFF                   jz      00403B94
00403E14   3B10                          cmp     edx, [eax]        //注册码是否为 "102332193"
00403E16   745C                          jz      00403E74
00403E18   3B08                          cmp     ecx, [eax]       ////注册码是否为 "30999945687"
00403E1A   740E                          jz      00403E2A
00403E1C   50                            push    eax
00403E1D   51                            push    ecx
```

* Reference to: `System.Proc_00403B94`

```asm
00403E1E   E871FDFFFF                    call    00403B94  //里面有 system.Move
                                             (void;void;void;void;Integer);

00403E23   5A                            pop     edx
00403E24   58                            pop     eax
```

* Reference to: `System.Proc_00403DC4`

```asm
00403E25   E99AFFFFFF                    jmp     00403DC4  //本段分析开始处
00403E2A   53                            push    ebx
00403E2B   56                            push    esi
00403E2C   57                            push    edi
00403E2D   89D3                          mov     ebx, edx
00403E2F   89CE                          mov     esi, ecx
00403E31   50                            push    eax
00403E32   8B43FC                        mov     eax, [ebx-4]
00403E35   0346FC                        add     eax, [esi-4]
```

* Reference to: `System.Proc_00403C00`

```asm
00403E38   E8C3FDFFFF                    call    00403C00
00403E3D   89C7                          mov     edi, eax
00403E3F   89C2                          mov     edx, eax
00403E41   89D8                          mov     eax, ebx
00403E43   8B4BFC                        mov     ecx, [ebx-4]
```

* Reference to: `system.Move(void;void;void;void;Integer);`

```asm
00403E46   E86DEAFFFF                    call    004028B8
00403E4B   89FA                          mov     edx, edi
00403E4D   89F0                          mov     eax, esi
00403E4F   8B4EFC                        mov     ecx, [esi-4]
00403E52   0353FC                        add     edx, [ebx-4]
```

* Reference to: `system.Move(void;void;void;void;Integer);`

```asm
00403E55   E85EEAFFFF                    call    004028B8
00403E5A   58                            pop     eax
00403E5B   89FA                          mov     edx, edi
00403E5D   85FF                          test    edi, edi
00403E5F   7403                          jz      00403E64
00403E61   FF4FF8                        dec     dword ptr [edi-8]
```

* Reference to: `System.Proc_00403B94`



### 00403E64 - 00403E76
```
00403E64   E82BFDFFFF           call   00403B94  
00403E69   5F                   pop    edi  
00403E6A   5E                   pop    esi  
00403E6B   5B                   pop    ebx  
00403E6C   C3                   ret  
00403E6D   89CA                 mov    edx, ecx  

* Reference to: `System.Proc_00403B94`  

00403E6F   E920FDFFFF           jmp    00403B94  
00403E74   89CA                 mov    edx, ecx  

* Reference to: `System.Proc_00403DC4`  

00403E76   E949FFFFFF           jmp    00403DC4    本段分析开始处  
00403E7B   C3                   ret  
```

============================================================================

注：这段代码用 `DEDE` 反汇编出来不正确，下面是 `OD` 反汇编的，在 `004AD1F7` 处相同，但是在 `DEDE` 反汇编出来的，下一句它的地址还是 `004AD1F7`。

### [分析五] `Switch (cases 1..15)`
```
004AD1D0    /$  55                   push ebp  
004AD1D1    |.  8BEC                 mov ebp, esp  
004AD1D3    |.  6A 00                push 0  
004AD1D5    |.  6A 00                push 0  
004AD1D7    |.  53                   push ebx                                  
//ebx=00000009  
004AD1D8    |.  56                   push esi                                  
//0000032A    用户名 ascii 累加和  
004AD1D9    |.  57                   push edi                                  
//0100B84C  
004AD1DA    |.  8BFA                 mov edi, edx                              
  
//0012FBC0  
004AD1DC    |.  8BD8                 mov ebx, eax                              
  
//00000063  "99"  
004AD1DE    |.  33C0                 xor eax, eax  
004AD1E0    |.  55                   push ebp  
004AD1E1    |.  68 3BD34A00          push talisman.004AD33B  
004AD1E6    |.  64:FF30              push dword ptr fs:[eax]  
004AD1E9    |.  64:8920              mov dword ptr fs:[eax], esp  
004AD1EC    |.  8BC3                 mov eax, ebx                              
  
//00000063  由"99"而来，固定了  
004AD1EE    |.  83F8 15              cmp eax,15                                    ;   

Switch (cases 1..15)  
004AD1F1    |.  0F87 EC000000        ja talisman.004AD2E3                        ；  
004AD1F7    |.  FF2485 FED14A00      jmp dword ptr ds:[eax*4+4AD1FE]  
004AD1FE    |.  E3D24A00             dd talisman.004AD2E3                        ;   

Switch table used at 004AD1F7  
004AD202    |.  56D24A00             dd talisman.004AD256  
004AD206    |.  60D24A00             dd talisman.004AD260  
004AD20A    |.  67D24A00             dd talisman.004AD267  
004AD20E    |.  6ED24A00             dd talisman.004AD26E  
004AD212    |.  75D24A00             dd talisman.004AD275  
004AD216    |.  7CD24A00             dd talisman.004AD27C  
004AD21A    |.  83D24A00             dd talisman.004AD283  
004AD21E    |.  8AD24A00             dd talisman.004AD28A  
004AD222    |.  91D24A00             dd talisman.004AD291  
004AD226    |.  98D24A00             dd talisman.004AD298  
004AD22A    |.  9CD24A00             dd talisman.004AD29C  
004AD22E    |.  A3D24A00             dd talisman.004AD2A3  
004AD232    |.  A7D24A00             dd talisman.004AD2A7  
004AD236    |.  AED24A00             dd talisman.004AD2AE  
004AD23A    |.  B5D24A00             dd talisman.004AD2B5  
004AD23E    |.  BCD24A00             dd talisman.004AD2BC  
004AD242    |.  C3D24A00             dd talisman.004AD2C3  
004AD246    |.  CAD24A00             dd talisman.004AD2CA  
004AD24A    |.  D1D24A00             dd talisman.004AD2D1  
004AD24E    |.  D8D24A00             dd talisman.004AD2D8  
004AD252    |.  DCD24A00             dd talisman.004AD2DC  
004AD256    |>  BE 3560A800          mov esi,0A86035                              ;   

Case 1 of switch 004AD1EE  
004AD25B    |.  E9 85000000          jmp talisman.004AD2E5  
004AD260    |>  BE EEF1C900          mov esi,0C9F1EE                              ;   

Case 2 of switch 004AD1EE  
004AD265    |.  EB 7E                jmp short talisman.004AD2E5  
004AD267    |>  BE 5460A800          mov esi,0A86054                              ;   

Case 3 of switch 004AD1EE  
004AD26C    |.  EB 77                jmp short talisman.004AD2E5  
004AD26E    |>  BE 453DDE00          mov esi,0DE3D45                              ;   

Case 4 of switch 004AD1EE  
004AD273    |.  EB 70                jmp short talisman.004AD2E5  
004AD275    |>  BE 9578CB00          mov esi,0CB7895                              ;   

Case 5 of switch 004AD1EE  
004AD27A    |.  EB 69                jmp short talisman.004AD2E5  
004AD27C    |>  BE 1D6CAB00          mov esi,0AB6C1D                              ;   

Case 6 of switch 004AD1EE  
004AD281    |.  EB 62                jmp short talisman.004AD2E5  
004AD283    |>  BE 37AE9900          mov esi,99AE37                               ;   

Case 7 of switch 004AD1EE  
004AD288    |.  EB 5B                jmp short talisman.004AD2E5  
004AD28A    |>  BE 4F7A1E01          mov esi,11E7A4F                              ;   

Case 8 of switch 004AD1EE  
004AD28F    |.  EB 54                jmp short talisman.004AD2E5  
004AD291    |>  BE 966CAB00          mov esi,0AB6C96                              ;   

Case 9 of switch 004AD1EE  
004AD296    |.  EB 4D                jmp short talisman.004AD2E5  
004AD298    |>  33F6                 xor esi,esi                                  ;   

Case A of switch 004AD1EE  
004AD29A    |.  EB 49                jmp short talisman.004AD2E5  
004AD29C    |>  BE 816DAB00          mov esi,0AB6D81                              ;   

Case B of switch 004AD1EE  
004AD2A1    |.  EB 42                jmp short talisman.004AD2E5  
004AD2A3    |>  33F6                 xor esi,esi                                  ;   

Case C of switch 004AD1EE  
004AD2A5    |.  EB 3E                jmp short talisman.004AD2E5  
004AD2A7    |>  BE E065DC00          mov esi,0DC65E0                              ;   

Case D of switch 004AD1EE  
004AD2AC    |.  EB 37                jmp short talisman.004AD2E5  
004AD2AE    |>  BE 9778CB00          mov esi,0CB7897                              ;   

Case E of switch 004AD1EE  
004AD2B3    |.  EB 30                jmp short talisman.004AD2E5  
004AD2B5    |>  BE BBADBA00          mov esi,0BAADBB                              ;   

Case F of switch 004AD1EE  
004AD2BA    |.  EB 29                jmp short talisman.004AD2E5  
004AD2BC    |>  BE 22F2AC00          mov esi,0ACF222                              ;   

Case 10 of switch 004AD1EE  
004AD2C1    |.  EB 22                jmp short talisman.004AD2E5  
004AD2C3    |>  BE 63812501          mov esi,1258163                              ;   

Case 11 of switch 004AD1EE  
004AD2C8    |.  EB 1B                jmp short talisman.004AD2E5  
004AD2CA    |>  BE 5EEFAF00          mov esi,0AFEF5E                              ;   

Case 12 of switch 004AD1EE  
004AD2CF    |.  EB 14                jmp short talisman.004AD2E5  
004AD2D1    |>  BE 1F922B01          mov esi,12B921F                              ;   

Case 13 of switch 004AD1EE  
004AD2D6    |.  EB 0D                jmp short talisman.004AD2E5  
004AD2D8    |>  33F6                 xor esi,esi                                  ;   

Case 14 of switch 004AD1EE  
004AD2DA    |.  EB 09                jmp short talisman.004AD2E5  
004AD2DC    |>  BE 6116F100          mov esi,0F11661                              ;   

Case 15 of switch 004AD1EE  
004AD2E1    |.  EB 02                jmp short talisman.004AD2E5  
```

跳来这:

```
004AD2E3    |>  \33F6                xor esi,esi                                  ;
```



## Default case of switch `004AD1EE`

```
004AD2E5    |>   83FB 15                  cmp ebx,15  

004AD2E8    |.   7E 0B                    jle short talisman.004AD2F5  

004AD2EA    |.   8BD7                     mov edx,edi                           

;0012FBC0  

004AD2EC    |.   8BC3                     mov eax,ebx                           

;00000063  

004AD2EE    |.   E8 59FDFFFF              call talisman.004AD04C                 ====>见下面[分析十]:  

004AD2F3    |.   EB 2B                    jmp short talisman.004AD320            回到这里，再跳  

004AD2F5    |>   \8D55 FC                 lea edx,dword ptr ss:[ebp-4]          ；jump from 004AD2E8  

004AD2F8    |.   8BC3                     mov eax,ebx  

004AD2FA    |.   E8 C9AFF5FF              call talisman.004082C8  

004AD2FF    |.   FF75 FC                  push dword ptr ss:[ebp-4]  

004AD302    |.   68 54D34A00              push talisman.004AD354  

004AD307    |.   8D55 F8                  lea edx,dword ptr ss:[ebp-8]  

004AD30A    |.   8BC6                     mov eax,esi  

004AD30C    |.   E8 B7AFF5FF              call talisman.004082C8  

004AD311    |.   FF75 F8                  push dword ptr ss:[ebp-8]  

004AD314    |.   8BC7                     mov eax,edi  

004AD316    |.   BA 03000000              mov edx,3  

004AD31B    |.   E8 5C6BF5FF              call talisman.00403E7C  

；跳到这里  

004AD320    |>   33C0                     xor eax,eax  

004AD322    |.   5A                         pop edx  

004AD323    |.   59                         pop ecx  

004AD324    |.   59                         pop ecx  

004AD325    |.   64:8910                    mov dword ptr fs:[eax],edx  

004AD328    |.   68 42D34A00              push talisman.004AD342  

004AD32D    |>   8D45 F8                  lea eax,dword ptr ss:[ebp-8]  

004AD330    |.   BA 02000000              mov edx,2  

004AD335    |.   E8 2A68F5FF              call talisman.00403B64  

004AD33A    \\.   C3                         retn                                //jmp 004AD342   

004AD33B     .^ E9 A462F5FF              jmp talisman.004035E4  

004AD340     .^ EB EB                     jmp short talisman.004AD32D  

004AD342     .  5F                         pop edi  

004AD343     .  5E                         pop esi  

004AD344     .  5B                         pop ebx  

004AD345     .  59                         pop ecx  

004AD346     .  59                         pop ecx  

004AD347     .  5D                         pop ebp  

004AD348     .  C3                         retn           ；回到 004ADADE  

\============================================================================================  

[分析六]Switch (cases 16..2A)  

004B4260    /$   55                         push ebp  

004B4261    |.   8BEC                         mov ebp,esp  

004B4263    |.   6A 00                        push 0  

004B4265    |.   6A 00                        push 0  

004B4267    |.   53                           push ebx  

004B4268    |.   56                           push esi  

004B4269    |.   57                           push edi  

004B426A    |.   8BF9                         mov edi,ecx  

004B426C    |.   8BF2                         mov esi,edx  

004B426E    |.   33C0                         xor eax,eax  

004B4270    |.   55                           push ebp  

004B4271    |.   68 C9434B00                  push talisman.004B43C9  

004B4276    |.   64:FF30                      push dword ptr fs:[eax]  

004B4279    |.   64:8920                      mov dword ptr fs:[eax],esp  

004B427C    |.   8BC6                         mov eax,esi  

004B427E    |.   83C0 EA                        add eax,-16                               ;   


```

## `Switch` (`cases 16..2A`)

```
004B4281    |.   83F8 14                        cmp eax,14  

004B4284    |.   0F87 F7000000                  ja talisman.004B4381  

004B428A    |.   FF2485 91424B00                jmp dword ptr ds:[eax*4+4B4291]  

004B4291    |.   EF424B00                       dd talisman.004B42EF                       ;   


```

### `Switch table used at 004B428A`

```
004B4295    |.   F9424B00                       dd talisman.004B42F9  

004B4299    |.   03434B00                       dd talisman.004B4303  

004B429D    |.   0A434B00                       dd talisman.004B430A  

004B42A1    |.   11434B00                       dd talisman.004B4311  

004B42A5    |.   18434B00                       dd talisman.004B4318  

004B42A9    |.   1F434B00                       dd talisman.004B431F  

004B42AD    |.   26434B00                       dd talisman.004B4326  

004B42B1    |.   2D434B00                       dd talisman.004B432D  

004B42B5    |.   34434B00                       dd talisman.004B4334  

004B42B9    |.   3B434B00                       dd talisman.004B433B  

004B42BD    |.   42434B00                       dd talisman.004B4342  

004B42C1    |.   49434B00                       dd talisman.004B4349  

004B42C5    |.   50434B00                       dd talisman.004B4350  

004B42C9    |.   57434B00                       dd talisman.004B4357  

004B42CD    |.   5E434B00                       dd talisman.004B435E  

004B42D1    |.   65434B00                       dd talisman.004B4365  

004B42D5    |.   6C434B00                       dd talisman.004B436C  

004B42D9    |.   73434B00                       dd talisman.004B4373  

004B42DD    |.   7A434B00                       dd talisman.004B437A  

004B42E1    |.   E5424B00                       dd talisman.004B42E5  

004B42E5    |>   BB F5034401                    mov ebx,14403F5                        ;   


```

### `Case 2A of switch 004B427E`

```
004B42EA    |.   E9 94000000                    jmp talisman.004B4383  

004B42EF    |>   BB DBB3A702                    mov ebx,2A7B3DB                        ;   


```

### `Case 16 of switch 004B427E`

```
004B42F4    |.   E9 8A000000                    jmp talisman.004B4383  

004B42F9    |>   BB 5A044401                    mov ebx,144045A                        ;   


```

### `Case 17 of switch 004B427E`

```
004B42FE    |.   E9 80000000                    jmp talisman.004B4383  

004B4303    |>   BB 9F55F502                    mov ebx,2F5559F                        ;   


```

### `Case 18 of switch 004B427E`

```
004B4308    |.   EB 79                          jmp short talisman.004B4383  

004B430A    |>   BB 357B5F01                    mov ebx,15F7B35                        ;   


```

### `Case 19 of switch 004B427E`

```
004B430F    |.   EB 72                          jmp short talisman.004B4383  

004B4311    |>   BB 71014401                    mov ebx,1440171                        ;   


```

### `Case 1A of switch 004B427E`

```
004B4316    |.   EB 6B                          jmp short talisman.004B4383  

004B4318    |>   BB 05A84401                    mov ebx,144A805                        ;   


```

### `Case 1B of switch 004B427E`

```
004B431D    |.   EB 64                          jmp short talisman.004B4383  

004B431F    |>   BB A528B204                    mov ebx,4B228A5                        ;   


```

### `Case 1C of switch 004B427E`

```
004B4324    |.   EB 5D                          jmp short talisman.004B4383  

004B4326    |>   BB 15B6DA01                    mov ebx,1DAB615                        ;   


```

### `Case 1D of switch 004B427E`

```
004B432B    |.   EB 56                          jmp short talisman.004B4383  

004B432D    |>   BB DB867601                    mov ebx,17686DB                        ;   


```

### `Case 1E of switch 004B427E`

```
004B4332    |.   EB 4F                          jmp short talisman.004B4383  

004B4334    |>   BB 81F43E04                    mov ebx,43EF481                        ;   


```

### `Case 1F of switch 004B427E`

```
004B4339    |.   EB 48                          jmp short talisman.004B4383  

004B433B    |>   BB D7F33E04                    mov ebx,43EF3D7                        ;   


```

### `Case 20 of switch 004B427E`

```
004B4340    |.   EB 41                          jmp short talisman.004B4383  

004B4342    |>   BB BFEE6F04                    mov ebx,46FEEBF                        ;   


```

### `Case 21 of switch 004B427E`

```
004B4347    |.   EB 3A                          jmp short talisman.004B4383  

004B4349    |>   BB A7363104                    mov ebx,43136A7                        ;   


```

### `Case 22 of switch 004B427E`

```
004B434E    |.   EB 33                          jmp short talisman.004B4383  

004B4350    |>   BB BB344E04                    mov ebx,44E34BB                        ;   


```

### `Case 23 of switch 004B427E`

```
004B4355    |.   EB 2C                          jmp short talisman.004B4383  



## Case 24 of switch 004B427E

```
004B4357    |>  BB F2034004            mov ebx,44003F2                         ;
```

## Case 25 of switch 004B427E

```
004B435C    |.  EB 25                  jmp short talisman.004B4383
```

## Case 26 of switch 004B427E

```
004B435E    |>  BB 6308B904           mov ebx,4B90863                         ;
```

## Case 27 of switch 004B427E

```
004B4363    |.  EB 1E                  jmp short talisman.004B4383
```

## Case 28 of switch 004B427E

```
004B4365    |>  BB E344EF02           mov ebx,2EF44E3                         ;
```

## Case 29 of switch 004B427E

```
004B436A    |.  EB 17                  jmp short talisman.004B4383
```

## Case 30 of switch 004B427E (Default)

```
004B436C    |>  BB 3B9EB503           mov ebx,3B59E3B                         ;
```

```
004B4371    |.  EB 10                  jmp short talisman.004B4383
```

```
004B4373    |>  BB 568F4604           mov ebx,4468F56                         ;
```

```
004B4378    |.  EB 09                  jmp short talisman.004B4383
```

```
004B437A    |>  BB 7BA28304           mov ebx,483A27B                         ;
```

```
004B437F    |.  EB 02                  jmp short talisman.004B4383
```

### Default case of switch 004B427E

```
004B4381    |>  33DB                   xor ebx,ebx                            ;
```

```
004B4383    |>  8D55 FC                lea edx,dword ptr ss:[ebp-4]
004B4386    |.  8BC6                   mov eax,esi
004B4388    |.  E8 3B3FF5FF            call talisman.004082C8
004B438D    |.  FF75 FC                push dword ptr ss:[ebp-4]
004B4390    |.  68 E0434B00            push talisman.004B43E0
004B4395    |.  8D55 F8                lea edx,dword ptr ss:[ebp-8]
004B4398    |.  8BC3                   mov eax,ebx
004B439A    |.  E8 293FF5FF            call talisman.004082C8
004B439F    |.  FF75 F8                push dword ptr ss:[ebp-8]
004B43A2    |.  8BC7                   mov eax,edi
004B43A4    |.  BA 03000000            mov edx,3
004B43A9    |.  E8 CEFAF4FF            call talisman.00403E7C
004B43AE    |.  33C0                   xor eax,eax
004B43B0    |.  5A                     pop edx
004B43B1    |.  59                     pop ecx
004B43B2    |.  59                     pop ecx
004B43B3    |.  64:8910                mov dword ptr fs:[eax],edx
004B43B6    |.  68 D0434B00            push talisman.004B43D0
004B43BB    |>  8D45 F8                lea eax,dword ptr ss:[ebp-8]
004B43BE    |.  BA 02000000            mov edx,2
004B43C3    |.  E8 9CF7F4FF            call talisman.00403B64
004B43C8    \.  C3                     retn
```

```
004B43C9      .^ E9 16F2F4FF            jmp talisman.004035E4
004B43CE      .^ EB EB                  jmp short talisman.004B43BB
```

```
004B43D0      .  5F                     pop edi
004B43D1      .  5E                     pop esi
004B43D2      .  5B                     pop ebx
004B43D3      .  59                     pop ecx
004B43D4      .  59                     pop ecx
004B43D5      .  5D                     pop ebp
004B43D6      .  C3                     retn
```

\===============================================================================

## [分析七] Switch (cases 2B..3F)

```
004BBCE0    /$  55                      push ebp
004BBCE1    |.  8BEC                    mov ebp,esp
004BBCE3    |.  6A 00                   push 0
004BBCE5    |.  6A 00                   push 0
004BBCE7    |.  53                      push ebx
004BBCE8    |.  56                      push esi
004BBCE9    |.  57                      push edi
004BBCEA    |.  8BF9                    mov edi,ecx
004BBCEC    |.  8BF2                    mov esi,edx
004BBCEE    |.  33C0                    xor eax,eax
004BBCF0    |.  55                      push ebp
004BBCF1    |.  68 43BE4B00             push talisman.004BBE43
004BBCF6    |.  64:FF30                 push dword ptr fs:[eax]
004BBCF9    |.  64:8920                 mov dword ptr fs:[eax],esp
004BBCFC    |.  8BC6                    mov eax,esi
004BBCFE    |.  83C0 D5                 add eax,-2B                             ;
```

### Switch (cases 2B..3F)

```
004BBD01    |.  83F8 14                 cmp eax,14
004BBD04    |.  0F87 F1000000           ja talisman.004BBDFB
004BBD0A    |.  FF2485 11BD4B00         jmp dword ptr ds:[eax*4+4BBD11]
```

### Switch table used at 004BBD0A

```
004BBD11    |.  79BD4B00                dd talisman.004BBD79                  ;
004BBD15    |.  80BD4B00                dd talisman.004BBD80
004BBD19    |.  87BD4B00                dd talisman.004BBD87
004BBD1D    |.  8BBD4B00                dd talisman.004BBD8B
004BBD21    |.  92BD4B00                dd talisman.004BBD92
004BBD25    |.  99BD4B00                dd talisman.004BBD99
004BBD29    |.  A0BD4B00                dd talisman.004BBDA0
004BBD2D    |.  A7BD4B00                dd talisman.004BBDA7
004BBD31    |.  AEBD4B00                dd talisman.004BBDAE
004BBD35    |.  B5BD4B00                dd talisman.004BBDB5
004BBD39    |.  BCBBD4B00               dd talisman.004BBDBC
004BBD3D    |.  C3BD4B00                dd talisman.004BBDC3
004BBD41    |.  CABD4B00                dd talisman.004BBDCA
004BBD45    |.  D1BD4B00                dd talisman.004BBDD1
004BBD49    |.  D8BD4B00                dd talisman.004BBDD8
004BBD4D    |.  DFBD4B00                dd talisman.004BBDDF
004BBD51    |.  E6BD4B00                dd talisman.004BBDE6
004BBD55    |.  EDBD4B00                dd talisman.004BBDED
004BBD59    |.  F4BD4B00                dd talisman.004BBDF4
004BBD5D    |.  65BD4B00                dd talisman.004BBD65
004BBD61    |.  6FBD4B00                dd talisman.004BBD6F
```

#### Case 3E ('>') of switch 004BBCFE

```
004BBD65    |>  BB 96F33E04             mov ebx,43EF396                         ;
004BBD6A    |.  E9 8E000000             jmp talisman.004BBDFD
```

#### Case 3F ('?') of switch 004BBCFE

```
004BBD6F    |>  BB F71EFB01             mov ebx,1FB1EF7                         ;
004BBD74    |.  E9 84000000             jmp talisman.004BBDFD
```

#### Case 2B ('+') of switch 004BBCFE

```
004BBD79    |>  BB F413DB01             mov ebx,1DB13F4                         ;
004BBD7E    |.  EB 7D                   jmp short talisman.004BBDFD
```

#### Case 2C (',') of switch 004BBCFE

```
004BBD80    |>  BB 456A0F02             mov ebx,20F6A45                         ;
004BBD85    |.  EB 76                   jmp short talisman.004BBDFD
```

#### Case 2D ('-') of switch 004BBCFE

```
004BBD87    |>  33DB                    xor ebx,ebx                             ;
004BBD89    |.  EB 72                   jmp short talisman.004BBDFD
```

#### Case 2E ('.') of switch 004BBCFE

```
004BBD8B    |>  BB 1D99DC01             mov ebx,1DC991D                         ;
004BBD90    |.  EB 6B                   jmp short talisman.004BBDFD
```

#### Case 2F ('/') of switch 004BBCFE

```
004BBD92    |>  BB 853EDD01             mov ebx,1DD3E85                         ;
004BBD97    |.  EB 64                   jmp short talisman.004BBDFD
```

#### Case 30 ('0') of switch 004BBCFE

```
004BBD99    |>  BB 4DCC4F02             mov ebx,24FCC4D                         ;
004BBD9E    |.  EB 5D                   jmp short talisman.004BBDFD
```

#### Case 31 ('1') of switch 004BBCFE

```
004BBDA0    |>  BB 9699DC01             mov ebx,1DC9996                         ;
004BBDA5    |.  EB 56                   jmp short talisman.004BBDFD
```

#### Case 32 ('2') of switch 004BBCFE

```
004BBDA7    |>  BB 45974003             mov ebx,3409745                         ;
004BBDAC    |.  EB 4F                   jmp short talisman.004BBDFD
```

#### Case 33 ('3') of switch 004BBCFE

```
004BBDAE    |>  BB 01317502             mov ebx,2753101                         ;
004BBDB3    |.  EB 48                   jmp short talisman.004BBDFD
```



### Case 34 ('4') of switch `004BBCFE`

```
004BBDB5    |>  BB 57307502            mov ebx,2753057                                     ;
```

### Case 35 ('5') of switch `004BBCFE`

```
004BBDBA    |.  EB 41                 jmp short talisman.004BBDFD
004BBDBC    |>  BB 3F2BA602           mov ebx,2A62B3F                                     ;
```

### Case 36 ('6') of switch `004BBCFE`

```
004BBDC1    |.  EB 3A                 jmp short talisman.004BBDFD
004BBDC3    |>  BB 173C9502           mov ebx,2953C17                                     ;
```

### Case 37 ('7') of switch `004BBCFE`

```
004BBDC8    |.  EB 33                 jmp short talisman.004BBDFD
004BBDCA    |>  BB 3B718402           mov ebx,284713B                                     ;
```

### Case 38 ('8') of switch `004BBCFE`

```
004BBDCF    |.  EB 2C                 jmp short talisman.004BBDFD
004BBDD1    |>  BB A2B57602           mov ebx,276B5A2                                     ;
```

### Case 39 ('9') of switch `004BBCFE`

```
004BBDD6    |.  EB 25                 jmp short talisman.004BBDFD
004BBDD8    |>  BB 856B0E03           mov ebx,30E6B85                                     ;
```

### Case 3A (':') of switch `004BBCFE`

```
004BBDDD    |.  EB 1E                 jmp short talisman.004BBDFD
004BBDDF    |>  BB 264E7302           mov ebx,2734E26                                     ;
```

### Case 3B (';') of switch `004BBCFE`

```
004BBDE4    |.  EB 17                 jmp short talisman.004BBDFD
004BBDE6    |>  BB B50BEB03           mov ebx,3EB0BB5                                     ;
```

### Case 3C ('<') of switch `004BBCFE`

```
004BBDEB    |.  EB 10                 jmp short talisman.004BBDFD
004BBDED    |>  BB 2ECE7C02           mov ebx,27CCE2E                                     ;
```

### Case 3D ('=') of switch `004BBCFE`

```
004BBDF2    |.  EB 09                 jmp short talisman.004BBDFD
004BBDF4    |>  BB 9E7C4304           mov ebx,4437C9E                                     ;
```

### Case 3D ('=') of switch `004BBCFE`

```
004BBDF9    |.  EB 02                 jmp short talisman.004BBDFD
004BBDFB    |>  33DB                  xor ebx,ebx                                       ;
```

### Default case of switch `004BBCFE`

```
004BBDFD    |>  8D55 FC               lea edx,dword ptr ss:[ebp-4]
004BBE00    |.  8BC6                  mov eax,esi
004BBE02    |.  E8 C1C4F4FF           call talisman.004082C8
004BBE07    |.  FF75 FC               push dword ptr ss:[ebp-4]
004BBE0A    |.  68 5CBE4B00           push talisman.004BBE5C
004BBE0F    |.  8D55 F8               lea edx,dword ptr ss:[ebp-8]
004BBE12    |.  8BC3                  mov eax,ebx
004BBE14    |.  E8 AFC4F4FF           call talisman.004082C8
004BBE19    |.  FF75 F8               push dword ptr ss:[ebp-8]
004BBE1C    |.  8BC7                  mov eax,edi
004BBE1E    |.  BA 03000000           mov edx,3
004BBE23    |.  E8 5480F4FF           call talisman.00403E7C
004BBE28    |.  33C0                  xor eax,eax
004BBE2A    |.  5A                    pop edx
004BBE2B    |.  59                    pop ecx
004BBE2C    |.  59                    pop ecx
004BBE2D    |.  64:8910               mov dword ptr fs:[eax],edx
004BBE30    |.  68 4ABE4B00           push talisman.004BBE4A
004BBE35    |>  8D45 F8               lea eax,dword ptr ss:[ebp-8]
004BBE38    |.  BA 02000000           mov edx,2
004BBE3D    |.  E8 227DF4FF           call talisman.00403B64
004BBE42    \\.  C3                   retn
004BBE43      .^ E9 9C77F4FF          jmp talisman.004035E4
004BBE48      .^ EB EB                jmp short talisman.004BBE35
004BBE4A      .  5F                   pop edi
004BBE4B      .  5E                   pop esi
004BBE4C      .  5B                   pop ebx
004BBE4D      .  59                   pop ecx
004BBE4E      .  59                   pop ecx
004BBE4F      .  5D                   pop ebp
004BBE50      .  C3                   retn
```

=============================================================

### [分析八]Switch (cases 40..54)

`call from 004ADB2C`

```
004ACECC    /$  55                    push ebp
004ACECD    |.  8BEC                  mov ebp,esp
004ACECF    |.  6A 00                 push 0
004ACED1    |.  6A 00                 push 0
004ACED3    |.  53                    push ebx
004ACED4    |.  56                    push esi
004ACED5    |.  57                    push edi
004ACED6    |.  8BFA                  mov edi,edx
004ACED8    |.  8BF0                  mov esi,eax
004ACEDA    |.  33C0                  xor eax,eax
004ACEDC    |.  55                    push ebp
004ACEDD    |.  68 2FD04A00           push talisman.004AD02F
004ACEE2    |.  64:FF30               push dword ptr fs:[eax]
004ACEE5    |.  64:8920               mov dword ptr fs:[eax],esp
004ACEE8    |.  8BC6                  mov eax,esi
004ACEEA    |.  83C0 C0               add eax,-40                                     ;
```

#### Switch (cases 40..54)

```
004ACEED    |.  83F8 14               cmp eax,14
004ACEF0    |.  0F87 F1000000          ja talisman.004ACFE7
004ACEF6    |.  FF2485 FDCE4A00        jmp dword ptr ds:[eax*4+4ACEFD]
004ACEFD    |.  6CCF4A00              dd talisman.004ACF6C                              ;
```

#### Switch table used at `004ACEF6`

```
004ACF01    |.  73CF4A00              dd talisman.004ACF73
004ACF05    |.  7ACF4A00              dd talisman.004ACF7A
004ACF09    |.  81CF4A00              dd talisman.004ACF81
004ACF0D    |.  88CF4A00              dd talisman.004ACF88
004ACF11    |.  8FCF4A00              dd talisman.004ACF8F
004ACF15    |.  96CF4A00              dd talisman.004ACF96
004ACF19    |.  9DCF4A00              dd talisman.004ACF9D
004ACF1D    |.  A4CF4A00              dd talisman.004ACFA4
004ACF21    |.  ABCF4A00              dd talisman.004ACFAB
004ACF25    |.  AFCF4A00              dd talisman.004ACFAF
004ACF29    |.  B6CF4A00              dd talisman.004ACFB6
004ACF2D    |.  BDCF4A00              dd talisman.004ACFBD
004ACF31    |.  C4CF4A00              dd talisman.004ACFC4
004ACF35    |.  CBCF4A00              dd talisman.004ACFCB
004ACF39    |.  D2CF4A00              dd talisman.004ACFD2
004ACF3D    |.  D9CF4A00              dd talisman.004ACFD9
004ACF41    |.  E0CF4A00              dd talisman.004ACFE0
004ACF45    |.  51CF4A00              dd talisman.004ACF51
004ACF49    |.  5BCF4A00              dd talisman.004ACF5B
004ACF4D    |.  65CF4A00              dd talisman.004ACF65
```

### Case 52 ('R') of switch `004ACEEA`

```
004ACF51    |>  BB 45520D03           mov ebx,30D5245                                     ;
004ACF56    |.  E9 8E000000           jmp talisman.004ACFE9
```

### Case 53 ('S') of switch `004ACEEA`

```
004ACF5B    |>  BB F74B2C03           mov ebx,32C4BF7                                     ;
004ACF60    |.  E9 84000000           jmp talisman.004ACFE9
```

### Case 54 ('T') of switch `004ACEEA`

```
004ACF65    |>  BB DAC70D03           mov ebx,30DC7DA                                     ;
004ACF6A    |.  EB 7D                 jmp short talisman.004ACFE9
```

### Case 40 ('@') of switch `004ACEEA`

```
004ACF6C    |>  BB 1796F704           mov ebx,4F79617                                     ;
004ACF71    |.  EB 76                 jmp short talisman.004ACFE9
```

### Case 41 ('A') of switch `004ACEEA`

```
004ACF73    |>  BB 95D22D03           mov ebx,32DD295                                     ;
004ACF78    |.  EB 6F                 jmp short talisman.004ACFE9
```

### Case 42 ('B') of switch `004ACEEA`

```
004ACF7A    |>  BB FFC50D03           mov ebx,30DC5FF                                     ;
004ACF7F    |.  EB 68                 jmp short talisman.004ACFE9
```

### Case 43 ('C') of switch `004ACEEA`

```
004ACF81    |>  BB 9EA97405           mov ebx,574A99E                                     ;
```



```
004ACF86    |.  EB 61                    jmp short talisman.004ACFE9  
004ACF88    |>  BB A5FB8003             mov ebx,380FBA5                     ;  
Case 44 ('D') of switch 004ACEEA  
004ACF8D    |.  EB 5A                    jmp short talisman.004ACFE9  
004ACF8F    |>  BB 96C60D03             mov ebx,30DC696                     ;  
Case 45 ('E') of switch 004ACEEA  
004ACF94    |.  EB 53                    jmp short talisman.004ACFE9  
004ACF96    |>  BB BB3FD103             mov ebx,3D13FBB                     ;  
Case 46 ('F') of switch 004ACEEA  
004ACF9B    |.  EB 4C                    jmp short talisman.004ACFE9  
004ACF9D    |>  BB 015EA603             mov ebx,3A65E01                     ;  
Case 47 ('G') of switch 004ACEEA  
004ACFA2    |.  EB 45                    jmp short talisman.004ACFE9  
004ACFA4    |>  BB 575DA603             mov ebx,3A65D57                     ;  
Case 48 ('H') of switch 004ACEEA  
004ACFA9    |.  EB 3E                    jmp short talisman.004ACFE9  
004ACFAB    |>  33DB                    xor ebx,ebx                         ;  
Case 49 ('I') of switch 004ACEEA  
004ACFAD    |.  EB 3A                    jmp short talisman.004ACFE9  
004ACFAF    |>  BB 1769C603             mov ebx,3C66917                     ;  
Case 4A ('J') of switch 004ACEEA  
004ACFB4    |.  EB 33                    jmp short talisman.004ACFE9  
004ACFB6    |>  BB 4F11BF04             mov ebx,4BF114F                     ;  
Case 4B ('K') of switch 004ACEEA  
004ACFBB    |.  EB 2C                    jmp short talisman.004ACFE9  
004ACFBD    |>  BB 5CE2A703             mov ebx,3A7E25C                     ;  
Case 4C ('L') of switch 004ACEEA  
004ACFC2    |.  EB 25                    jmp short talisman.004ACFE9  
004ACFC4    |>  BB 39766201             mov ebx,1627639                     ;  
Case 4D ('M') of switch 004ACEEA  
004ACFC9    |.  EB 1E                    jmp short talisman.004ACFE9  
004ACFCB    |>  BB F0B6B002             mov ebx,2B0B6F0                     ;  
Case 4E ('N') of switch 004ACEEA  
004ACFD0    |.  EB 17                    jmp short talisman.004ACFE9  
004ACFD2    |>  BB 25BF4A05             mov ebx,54ABF25                     ;  
Case 4F ('O') of switch 004ACEEA  
004ACFD7    |.  EB 10                    jmp short talisman.004ACFE9  
004ACFD9    |>  BB EEB89E03             mov ebx,39EB8EE                     ;  
Case 50 ('P') of switch 004ACEEA  
004ACFDE    |.  EB 09                    jmp short talisman.004ACFE9  
004ACFE0    |>  BB F882EA03             mov ebx,3EA82F8                     ;  
Case 51 ('Q') of switch 004ACEEA  
004ACFE5    |.  EB 02                    jmp short talisman.004ACFE9  
004ACFE7    |>  33DB                    xor ebx,ebx                         ;  
Default case of switch 004ACEEA  
004ACFE9    |>  8D55 FC                 lea edx,dword ptr ss:\[ebp-4\]  
004ACFEC    |.  8BC6                    mov eax,esi  
004ACFEE    |.  E8 D5B2F5FF             call talisman.004082C8  
004ACFF3    |.  FF75 FC                 push dword ptr ss:\[ebp-4\]  
004ACFF6    |.  68 48D04A00             push talisman.004AD048  
004ACFFB    |.  8D55 F8                 lea edx,dword ptr ss:\[ebp-8\]  
004ACFFE    |.  8BC3                    mov eax,ebx  
004AD000    |.  E8 C3B2F5FF             call talisman.004082C8  
004AD005    |.  FF75 F8                 push dword ptr ss:\[ebp-8\]  
004AD008    |.  8BC7                    mov eax,edi  
004AD00A    |.  BA 03000000             mov edx,3  
004AD00F    |.  E8 686EF5FF             call talisman.00403E7C  
004AD014    |.  33C0                    xor eax,eax  
004AD016    |.  5A                      pop edx  
004AD017    |.  59                      pop ecx  
004AD018    |.  59                      pop ecx  
004AD019    |.  64:8910                 mov dword ptr fs:\[eax\],edx  
004AD01C    |.  68 36D04A00             push talisman.004AD036  
004AD021    |>  8D45 F8                 lea eax,dword ptr ss:\[ebp-8\]  
004AD024    |.  BA 02000000             mov edx,2  
004AD029    |.  E8 366BF5FF             call talisman.00403B64  
004AD02E    \\.  C3                     retn  
004AD02F     .^ E9 B065F5FF             jmp talisman.004035E4  
004AD034     .^ EB EB                   jmp short talisman.004AD021  
004AD036     .  5F                      pop edi  
004AD037     .  5E                      pop esi  
004AD038     .  5B                      pop ebx  
004AD039     .  59                      pop ecx  
004AD03A     .  59                      pop ecx  
004AD03B     .  5D                      pop ebp  
004AD03C     .  C3                      retn  

=============================================================================================

### [分析九]--比较字符串

00403ECC   53                 push    ebx  
00403ECD   56                 push    esi  
00403ECE   57                 push    edi  
00403ECF   89C6               mov     esi, eax   //ESI 00FD1BEC ASCII "21933099" str1  
00403ED1   89D7               mov     edi, edx   //EDX 00FD1C1C ASCII "100338181" str2  
00403ED3   39D0               cmp     eax, edx  
00403ED5   0F848F000000       jz      00403F6A  
00403EDB   85F6               test    esi, esi  
00403EDD   7468               jz      00403F47  
00403EDF   85FF               test    edi, edi  
00403EE1   746B               jz      00403F4E  
00403EE3   8B46FC             mov     eax, \[esi-$04\] //eax=00000008，"21933099"的长度  
00403EE6   8B57FC             mov     edx, \[edi-$04\] //eax=00000009  "100338181"的长度  
00403EE9   29D0               sub     eax, edx     
00403EEB   7702               jnbe    00403EEF     // str1 的长度小于 str2 的长度，跳；  
str1=str2  
00403EED   01C2               add     edx, eax     //edx 为 str1 的长度  
00403EEF   52                 push    edx  
00403EF0   C1EA02             shr     edx, $02  
00403EF3   7426               jz      00403F1B  
00403EF5   8B0E               mov     ecx, \[esi\]  
00403EF7   8B1F               mov     ebx, \[edi\]  
00403EF9   39D9               cmp     ecx, ebx  
00403EFB   7558               jnz     00403F55  
00403EFD   4A                 dec     edx  
00403EFE   7415               jz      00403F15  
* Reference to field `TAboutBox.Owner` : `TComponent`  
|  
Begin LOOP：  
00403F00   8B4E04             mov     ecx, \[esi+$04\] //str1  
00403F03   8B5F04             mov     ebx, \[edi+$04\] //str2  
00403F06   39D9               cmp     ecx, ebx       //不等就跳走，必须相同  
00403F08   754B               jnz     00403F55  
00403F0A   83C608             add     esi, +$08  
00403F0D   83C708             add     edi, +$08  
00403F10   4A                 dec     edx  
00403F11   75E2               jnz     00403EF5  
END LOOP  
00403F13   EB06               jmp     00403F1B  
00403F15   83C604             add     esi, +$04  
00403F18   83C704             add     edi, +$04  
00403F1B   5A                 pop     edx  
00403F1C   83E203             and     edx, +$03  
00403F1F   7422               jz      00403F43  
00403F21   8B0E               mov     ecx, \[esi\]  
00403F23   8B1F               mov     ebx, \[edi\]  
00403F25   38D9               cmp     cl, bl  
00403F27   7541               jnz     00403F6A  
00403F29   4A                 dec     edx  
00403F2A   7417               jz      00403F43  
00403F2C   38FD               cmp     ch, bh  
00403F2E   753A               jnz     00403F6A  
00403F30   4A                 dec     edx  
00403F31   7410               jz      00403F43  
```



# 汇编代码片段

```
00403F33   81E30000FF00           and     ebx, $00FF0000  
00403F39   81E10000FF00           and     ecx, $00FF0000  
00403F3F   39D9                   cmp     ecx, ebx  
00403F41   7527                   jnz     00403F6A  
00403F43   01C0                   add     eax, eax  
00403F45   EB23                   jmp     00403F6A  
00403F47   8B57FC                 mov     edx, [edi-$04]  
00403F4A   29D0                   sub     eax, edx  
00403F4C   EB1C                   jmp     00403F6A  
00403F4E   8B46FC                 mov     eax, [esi-$04]  
00403F51   29D0                   sub     eax, edx  
00403F53   EB15                   jmp     00403F6A  
00403F55   5A                     pop     edx  
00403F56   38D9                   cmp     cl, bl  
00403F58   7510                   jnz     00403F6A  
00403F5A   38FD                   cmp     ch, bh  
00403F5C   750C                   jnz     00403F6A  
00403F5E   C1E910                 shr     ecx, $10  
00403F61   C1EB10                 shr     ebx, $10  
00403F64   38D9                   cmp     cl, bl  
00403F66   7502                   jnz     00403F6A  
00403F68   38FD                   cmp     ch, bh  
00403F6A   5F                     pop     edi  
00403F6B   5E                     pop     esi  
00403F6C   5B                     pop     ebx  
00403F6D   C3                     ret  
```

====================================================

## [分析十]Switch (cases 55..69)

from `004AD2EE`  `{call 004AD04C}`

```
004AD04C    /$  55                    push ebp  
004AD04D    |.  8BEC                  mov ebp,esp  
004AD04F    |.  6A 00                 push 0  
004AD051    |.  6A 00                 push 0  
004AD053    |.  53                    push ebx                     ;00000063  
004AD054    |.  56                    push esi                     ;00000000  
004AD055    |.  57                    push edi                     ;0012FBC0  
004AD056    |.  8BFA                 mov edi,edx                   ;0012FBC0  
004AD058    |.  8BF0                 mov esi,eax                   ;00000063  
004AD05A    |.  33C0                 xor eax,eax  
004AD05C    |.  55                    push ebp  
004AD05D    |.  68 B5D14A00           push talisman.004AD1B5  
004AD062    |.  64:FF30              push dword ptr fs:[eax]  
004AD065    |.  64:8920              mov dword ptr fs:[eax],esp  
004AD068    |.  8BC6                 mov eax,esi                               
  
;00000063  
004AD06A    |.  83C0 AB              add eax,-55                                  ;   
Switch (cases 55..69)   eax=0000000E  
004AD06D    |.  83F8 14              cmp eax,14  
004AD070    |.  0F87 F7000000        ja talisman.004AD16D  
004AD076    |.  FF2485 7DD04A00     jmp dword ptr ds:[eax*4+4AD07D]  
004AD07D    |. /F6D04A00             dd talisman.004AD0F6                            ;   
Switch table used at 004AD076  
004AD081    |. |FDD04A00             dd talisman.004AD0FD  
004AD085    |. |04D14A00             dd talisman.004AD104  
004AD089    |. |0BD14A00             dd talisman.004AD10B  
004AD08D    |. |12D14A00             dd talisman.004AD112  
004AD091    |. |19D14A00             dd talisman.004AD119  
004AD095    |. |20D14A00             dd talisman.004AD120  
004AD099    |. |27D14A00             dd talisman.004AD127  
004AD09D    |. |2ED14A00             dd talisman.004AD12E  
004AD0A1    |. |35D14A00             dd talisman.004AD135  
004AD0A5    |. |3CD14A00             dd talisman.004AD13C  
004AD0A9    |. |43D14A00             dd talisman.004AD143  
004AD0AD    |. |4AD14A00             dd talisman.004AD14A  
004AD0B1    |. |51D14A00             dd talisman.004AD151  
004AD0B5    |. |58D14A00             dd talisman.004AD158  
004AD0B9    |. |5FD14A00             dd talisman.004AD15F  
004AD0BD    |. |66D14A00             dd talisman.004AD166  
004AD0C1    |. |D1D04A00             dd talisman.004AD0D1  
004AD0C5    |. |DBD04A00             dd talisman.004AD0DB  
004AD0C9    |. |E5D04A00             dd talisman.004AD0E5  
004AD0CD    |. |EFD04A00             dd talisman.004AD0EF  
004AD0D1    |> |BB B98AD704          mov ebx,4D78AB9                            ;  Case  
66 ('f') of switch 004AD06A  
004AD0D6    |. |E9 94000000          jmp talisman.004AD16F  
004AD0DB    |> |BB 770FF604          mov ebx,4F60F77                            ;  Case  
67 ('g') of switch 004AD06A  
004AD0E0    |. |E9 8A000000          jmp talisman.004AD16F  
004AD0E5    |> |BB 5A8BD704          mov ebx,4D78B5A                            ;  Case  
68 ('h') of switch 004AD06A  
004AD0EA    |. |E9 80000000          jmp talisman.004AD16F  
004AD0EF    |> |BB C55A0A05          mov ebx,50A5AC5                            ;  Case  
69 ('i') of switch 004AD06A  
004AD0F4    |. |EB 79                jmp short talisman.004AD16F  
004AD0F6    |> \\BB 1596F704          mov ebx,4F79615                           ;  Case  
55 ('U') of switch 004AD06A  
004AD0FB    |.  EB 72                jmp short talisman.004AD16F  
004AD0FD    |>  BB 9D89D704          mov ebx,4D7899D                            ;  Case  
56 ('V') of switch 004AD06A  
004AD102    |.  EB 6B                jmp short talisman.004AD16F  
004AD104    |>  BB 052FD804          mov ebx,4D82F05                            ;  Case  
57 ('W') of switch 004AD06A  
004AD109    |.  EB 64                jmp short talisman.004AD16F  
004AD10B    |>  BB 77782604          mov ebx,4267877                            ;  Case  
58 ('X') of switch 004AD06A  
004AD110    |.  EB 5D                jmp short talisman.004AD16F  
004AD112    |>  BB D67CD404          mov ebx,4D47CD6                            ;  Case  
59 ('Y') of switch 004AD06A  
004AD117    |.  EB 56                jmp short talisman.004AD16F  
004AD119    |>  BB BB6C0205          mov ebx,5026CBB                            ;  Case  
5A ('Z') of switch 004AD06A  
004AD11E    |.  EB 4F                jmp short talisman.004AD16F  
004AD120    |>  BB 018BD704          mov ebx,4D78B01                            ;  Case  
5B ('[') of switch 004AD06A  
004AD125    |.  EB 48                jmp short talisman.004AD16F  
004AD127    |>  BB 4D8AD704          mov ebx,4D78A4D                            ;  Case  
5C ('') of switch 004AD06A  
004AD12C    |.  EB 41                jmp short talisman.004AD16F  
004AD12E    |>  BB 3F850805          mov ebx,508853F                            ;  Case  
5D (']') of switch 004AD06A  
004AD133    |.  EB 3A                jmp short talisman.004AD16F  
004AD135    |>  BB 5B5E2004          mov ebx,4205E5B                            ;  Case  
5E ('^') of switch 004AD06A  
004AD13A    |.  EB 33                jmp short talisman.004AD16F  
004AD13C    |>  BB 3BCBE604          mov ebx,4E6CB3B                            ;  Case  
5F ('_') of switch 004AD06A  
004AD141    |.  EB 2C                jmp short talisman.004AD16F  
004AD143    |>  BB 22A67105          mov ebx,571A622                            ;  Case  
60 ('`') of switch 004AD06A  
004AD148    |.  EB 25                jmp short talisman.004AD16F  
004AD14A    |>  BB 2328E705          mov ebx,5E72823                            ;  Case  
61 ('a') of switch 004AD06A  
004AD14F    |.  EB 1E                jmp short talisman.004AD16F  
004AD151    |>  BB 1EE6AA03          mov ebx,3AAE61E                            ;  Case  
62 ('b') of switch 004AD06A  
004AD156    |.  EB 17                jmp short talisman.004AD16F  
跳到这里:  
004AD158    |> \\BB 1F46F005          mov ebx,5F0461F                           ;  Case  
63 ('c') of switch 004AD06A,这是个常数,"99632671"  
004AD15D    |.  EB 10                jmp short talisman.004AD16F  
004AD15F    |> \\BB D6F8AD03          mov ebx,3ADF8D6                           ;  Case  
64 ('d') of switch 004AD06A
```



# 004AD164 - Talisman Disassembly

```
004AD164    |.  EB 09              jmp short talisman.004AD16F

004AD166    |>  BB C5D37601        mov ebx,176D3C5                     ;  Case

65 ('e') of switch 004AD06A

004AD16B    |.  EB 02              jmp short talisman.004AD16F

004AD16D    |>  33DB              xor ebx,ebx                         ;

Default case of switch 004AD06A

004AD16F    |>  8D55 FC           lea edx,dword ptr ss:[ebp-4]

;0012FA64

004AD172    |.  8BC6              mov eax,esi

;00000063

004AD174    |.  E8 4FB1F5FF      call talisman.004082C8               ;见前面

的分析,转化为"99"

004AD179    |.  FF75 FC          push dword ptr ss:[ebp-4]

004AD17C    |.  68 CCD14A00      push talisman.004AD1CC

004AD181    |.  8D55 F8          lea edx,dword ptr ss:[ebp-8]

004AD184    |.  8BC3             mov eax,ebx

004AD186    |.  E8 3DB1F5FF     call talisman.004082C8               ；

004AD18B    |.  FF75 F8         push dword ptr ss:[ebp-8]

004AD18E    |.  8BC7             mov eax,edi

004AD190    |.  BA 03000000      mov edx,3

004AD195    |.  E8 E26CF5FF     call talisman.00403E7C               ；F8

004AD19A    |.  33C0             xor eax,eax

004AD19C    |.  5A               pop edx

004AD19D    |.  59               pop ecx

004AD19E    |.  59               pop ecx

004AD19F    |.  64:8910          mov dword ptr fs:[eax],edx

004AD1A2    |.  68 BCD14A00     push talisman.004AD1BC

004AD1A7    |>  8D45 F8          lea eax,dword ptr ss:[ebp-8]

004AD1AA    |.  BA 02000000     mov edx,2

004AD1AF    |.  E8 B069F5FF     call talisman.00403B64

004AD1B4    \\. C3               retn                                 ；跳到 004AD1BC

004AD1B5     .^ E9 2A64F5FF      jmp talisman.004035E4

004AD1BA     .^ EB EB            jmp short talisman.004AD1A7

004AD1BC     .  5F               pop edi

004AD1BD     .  5E               pop esi

004AD1BE     .  5B               pop ebx

004AD1BF     .  59               pop ecx

004AD1C0     .  59               pop ecx

004AD1C1     .  5D               pop ebp

004AD1C2     .  C3               retn                                 ；回到 004AD2F3
```

=============================================================================================
=

## 【破解总结】

首先，我们输入的注册码为`"1234567890"`，长度为`10`，因而在`[分析一]004896FA`处没有跳。

接下来，程序帮我们生成了一个长为`20`的注册码`"10233219330999945687"`，放心，这个码肯定是不能通过验证的，而且很具有迷惑性。

`"30999945687"`为程序中的`64`位十六进制常数`737BDA1D7`的十进制数字串，`"102332193"`则为常数`$06197721`的十进制数字串。

接下来程序又计算了用户名的`ascii`码的累加和。对注册码的处理是：`10233 21933099 99 45687`取出了`14`，`15`位组成二位十进制数。取出第六位开始的八个，组成八位十进制数，称之为`regcode1`。

然后根据那个两位十进制数，再来查表，确定一个姑且称之为`basecode`的数，这个`basecode + 用户名的累加和 * $00000367 = regcode2`。

最后`if (regcode1 == regcode2)` `register success!`

由于我们的注册码长度为`10`，因而`"10233219330999945687"`是常数，则`basecode`也是常数，`regcode1`也是常数，我们反求用户名`ascii`码累加和：

`(regcode1 - basecode) DIV $00000367`，而`regcode1 - basecode < 0`，故这是一个永假的判断，即不管用户名（由`ascii`码组成，汉字没验证，不敢说）是什么，总会报错。

跟踪时发现注册码只能为数字，注册码长度不小于`20`时跳过了一段程序。那么，现在我们就改变注册码长度为`20`，或更长，再来跟踪看看。

在注册码窗口中输入`20`位后，便不能再输入。看来用户名长度必为`20`。`regcode: "12345678901234567890"`。唯一的差别是跳过了一段生成`"10233219330999945687"`的代码，取而代之的是我们自己输入的注册码，看来一切都清楚了。

```
1 2 3 4 5  6 7 8 9 a b c d e f 10 11 12 13 14
              + + + + + + + + * *
CNwinndy(ascii sum=032A)032A*0367=000AC3E6
```

取两位十进制数为`24`来手动计算一下看看：

```
24
2F5559F + 000AC3E6 = 3001985(50338181)
xxxxx5033818124xxxxx.注册成功！x 代表任意十进制个位数。
```

取为`07`(个位数)再看看：

```
07
99AE37 + 000AC3E6 = A4721D(10777117)
xxxxx1077711707xxxxx 注册成功!
```

那么，注册信息保存在哪里？

搜索`CNwinndy`，`xxxxx5033818124xxxxx`，都没找到，看来不是用明文保存的。

还记得`HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced`下的`WCID`键吧，打开看看，发现里面有`32`行，每行`8`个`byte`的数据，咦？前面跟踪的时候，见`004896CE`处初始化了一个数组的，`256`个`byte`的数组，在`00489792`处把版本信息`"2810"`也存在里面了，注册信息肯定也在其中了，删除`WCID`这个键再打开程序看看，删除，再运行，变成没注册的版本！`OK`！

尽管，用户名在中途经过变换，但是在计算`ascii`和的时候，程序还是还原了。程序这样做，使得在调试时容易跟丢，因为不是明文了。

但是，程序还是还原了，然后再计算和，如果我们不还原，而是在计算变换后的码的和`sum1`，然后再加上`length(UserName) * 0F`(还原的方式是加上`0F`)，这样也可得到`sum`，这样就更具有迷惑性了！

### 注册机的编写

可以先定下两位十进制数(个位数前面加`0`)，用作注册码的`14`，`15`位，然后根据这两位十进制数查表，得到`basecode`，然后计算出注册码的第`6`位-`14`位，设为`regcode2`，前面五位和后面五位可以取任意十进制数字。如果定下的两位十进制数，使得`regcode1`，必须保证`regcode2`为八位十进制数。`10000000(989680)-99999999(5F5E0FF)`长度不为`8`，则需要调整这两位十进制数。通过查表发现所有的`basecode`都比`10000000(989680)`大，看来长度不会小于`8`，只可能大于`8`了。若长度大于八，则我们就需要调整两位十进制数，重新确定`basecode`，以使得`regcode2`长度为八。

但从程序内定的两位数字和`basecode`的对应关系来看，似乎没有规律，可见，用户名与这两位十进制数之间有一定的联系，它们之间是多对多的关系。选择两位数字时，有三种方案：

1.  可随机生成`01-99`之间的两位数，然后计算`regcode2`，然后检查`LENGTH(regcode2)`，`if LENGTH(regcode2) <> 8`，再随机生成两位数。直到符合要求。不会出现死循环。但是注册码不具有再现性，好像也没必要哦。
2.  按某种优先级别来依次选择`01-99`间的数，根据用户名累加和出现的频率，确定最通用的`basecode`(可能不止一个)，然后确定出优先级比较靠前的二位数。这样的话，减少了查表时间，效率高。但哪个`basecode`使用频率高可能不太容易确定。
3.  或者根据用户名累加和来`hash`一下(但这个`hash`函数要选好...，似乎不太合适)。

综上比较，觉得随机生成那两位十进制数的方案比较好，考虑到长用户名的情况很少，如果随机生成的两位数是`0A,0C,14,2D,49`，则重新生成随机数字。`case A,C,14,2D,49`，这四种情况下的`basecode`为`0`，适用于用户名很长的情况，这个时候仅用户名累加和的十进制串就可以达到八位数。

注册机伪码为：

```
begin KeyGen_Talisman(string userName)
    int LuserNmae;
    static LongInt basecode(99)={...这里初始化表格}；
    LuserNmae= GetLength(userName);//用户名长度
    if LuserNmae==0 exitsub；//用户名为空，则退出程序
    LongInt sum=0;
    for(int i=0;i<LuserNmae;i++) //计算用户名的 ascii 累加和
        sum+=GetAsc(MID(userName,i,1));
    int index;//注册码的第十四十五组成的两位数
    do{
        if(HexToLongInt(sum)>99999999) then  index=RandomSelect(0A,0C,14,2D,49);//随机选择
        else
            {do
            Index=GetRandom();0<tempIndex<100
            while( Index==0x0A OR Index==0x0C OR Index==0x14 OR Index==0x2D OR
Index==0x49);
            }
LongInt regcode1,temp1;
string temp2;
regcode1=basecode(index)+sum*0x367;
temp1=Hexto10(regcode1);
temp2=Int2Str(temp1);
}while(GetLength(temp2)<>8);//regcode 的长度不是 8(>8)则回过头重新随机生成两位数
regcode="xxxxx"+temp2+Int2Str(index)+"XXXXX";
ShowRegCode(regcode,'X'可以为任意十进制个位数);
end
```

下面是整理出来的表：

```
mov esi,0A86035                     ;  Case 1 of switch 004AD1EE
mov esi,0C9F1EE                     ;  Case 2 of switch 004AD1EE
mov esi,0A86054                     ;  Case 3 of switch 004AD1EE
mov esi,0DE3D45                     ;  Case 4 of switch 004AD1EE
mov esi,0CB7895                     ;  Case 5 of switch 004AD1EE
mov esi,0AB6C1D                     ;  Case 6 of switch 004AD1EE
mov esi,99AE37                      ;  Case 7 of switch 004AD1EE
mov esi,11E7A4F                     ;  Case 8 of switch 004AD1EE
mov esi,0AB6C96                     ;  Case 9 of switch 004AD1EE
xor esi,esi                         ;  Case A of switch 004AD1EE

**************
mov esi,0AB6D81                     ;  Case B of switch 004AD1EE
xor esi,esi                         ;  Case C of switch 004AD1EE

**************
mov esi,0DC65E0                     ;  Case D of switch 004AD1EE
mov esi,0CB7897                     ;  Case E of switch 004AD1EE
mov esi,0BAADBB                     ;  Case F of switch 004AD1EE
mov esi,0ACF222                     ;  Case 10 of switch 004AD1EE
```



```asm
  mov esi,1258163                              ; Case 11 of switch 004AD1EE

  mov esi,0AFEF5E                              ; Case 12 of switch 004AD1EE

  mov esi,12B921F                              ; Case 13 of switch 004AD1EE

 xor esi,esi                                    ; Case 14 of switch 004AD1EE

****

 mov esi,0F11661                               ; Case 15 of switch 004AD1EE

  mov ebx,2A7B3DB                              ; Case 16 of switch 004B427E
  mov ebx,144045A                              ; Case 17 of switch 004B427E
  mov ebx,2F5559F                              ; Case 18 of switch 004B427E
  mov ebx,15F7B35                              ; Case 19 of switch 004B427E

//revenge 组织的 Nothing 写了个注册机，
                            //我随机输
一些用户名检测，发现他的两位数定为 25

  mov ebx,1440171                              ; Case 1A of switch 004B427E
  mov ebx,144A805                              ; Case 1B of switch 004B427E
   mov ebx,4B228A5                              ; Case 1C of switch 004B427E
  mov ebx,1DAB615                              ; Case 1D of switch 004B427E
  mov ebx,17686DB                              ; Case 1E of switch 004B427E
   mov ebx,43EF481                              ; Case 1F of switch 004B427E
   mov ebx,43EF3D7                              ; Case 20 of switch 004B427E
  mov ebx,46FEEBF                              ; Case 21 of switch 004B427E
   mov ebx,43136A7                              ; Case 22 of switch 004B427E
   mov ebx,44E34BB                              ; Case 23 of switch 004B427E
     mov ebx,44003F2                              ; Case 24 of switch 004B427E
  mov ebx,4B90863                              ; Case 25 of switch 004B427E
   mov ebx,2EF44E3                              ; Case 26 of switch 004B427E
   mov ebx,3B59E3B                              ; Case 27 of switch 004B427E
  mov ebx,4468F56                              ; Case 28 of switch 004B427E
  mov ebx,483A27B                              ; Case 29 of switch 004B427E
 mov ebx,14403F5                              ; Case 2A of switch 004B427E
  mov ebx,1DB13F4                              ; Case 2B ('+') of switch 004BBCFE
 mov ebx,20F6A45                              ; Case 2C (',') of switch 004BBCFE
  xor ebx,ebx                                    ; Case 2D ('-') of switch 004BBCFE

****
   mov ebx,1DC991D                              ; Case 2E ('.') of switch 004BBCFE
  mov ebx,1DD3E85                              ; Case 2F ('/') of switch 004BBCFE
   mov ebx,24FCC4D                              ; Case 30 ('0') of switch 004BBCFE
 mov ebx,1DC9996                              ; Case 31 ('1') of switch 004BBCFE
 mov ebx,3409745                              ; Case 32 ('2') of switch 004BBCFE
  mov ebx,2753101                              ; Case 33 ('3') of switch 004BBCFE
 mov ebx,2753057                              ; Case 34 ('4') of switch 004BBCFE
  mov ebx,2A62B3F                              ; Case 35 ('5') of switch 004BBCFE
  mov ebx,2953C17                              ; Case 36 ('6') of switch 004BBCFE
  mov ebx,284713B                              ; Case 37 ('7') of switch 004BBCFE
   mov ebx,276B5A2                              ; Case 38 ('8') of switch 004BBCFE
     mov ebx,30E6B85                              ; Case 39 ('9') of switch 004BBCFE
    mov ebx,2734E26                              ; Case 3A (':') of switch 004BBCFE
   mov ebx,3EB0BB5                              ; Case 3B (';') of switch 004BBCFE
   mov ebx,27CCE2E                              ; Case 3C ('<') of switch 004BBCFE
   mov ebx,4437C9E                              ; Case 3D ('=') of switch 004BBCFE
    mov ebx,43EF396                              ; Case 3E ('>') of switch 004BBCFE
 mov ebx,1FB1EF7                              ; Case 3F ('?') of switch 004BBCFE
 mov ebx,4F79617                              ; Case 40 ('@') of switch 004ACEEA
   mov ebx,32DD295                              ; Case 41 ('A') of switch 004ACEEA
   mov ebx,30DC5FF                              ; Case 42 ('B') of switch 004ACEEA
 mov ebx,574A99E                              ; Case 43 ('C') of switch 004ACEEA
  mov ebx,380FBA5                              ; Case 44 ('D') of switch 004ACEEA
 mov ebx,30DC696                              ; Case 45 ('E') of switch 004ACEEA
     mov ebx,3D13FBB                              ; Case 46 ('F') of switch 004ACEEA
   mov ebx,3A65E01                              ; Case 47 ('G') of switch 004ACEEA
 mov ebx,3A65D57                              ; Case 48 ('H') of switch 004ACEEA
  xor ebx,ebx                                    ; Case 49 ('I') of switch 004ACEEA

****
 mov ebx,3C66917                              ; Case 4A ('J') of switch 004ACEEA
  mov ebx,4BF114F                              ; Case 4B ('K') of switch 004ACEEA
 mov ebx,3A7E25C                              ; Case 4C ('L') of switch 004ACEEA
   mov ebx,1627639                              ; Case 4D ('M') of switch 004ACEEA
  mov ebx,2B0B6F0                              ; Case 4E ('N') of switch 004ACEEA
 mov ebx,54ABF25                              ; Case 4F ('O') of switch 004ACEEA
   mov ebx,39EB8EE                              ; Case 50 ('P') of switch 004ACEEA
  mov ebx,3EA82F8                              ; Case 51 ('Q') of switch 004ACEEA
    mov ebx,30D5245                              ; Case 52 ('R') of switch 004ACEEA
  mov ebx,32C4BF7                              ; Case 53 ('S') of switch 004ACEEA
 mov ebx,30DC7DA                              ; Case 54 ('T') of switch 004ACEEA
  mov ebx,4F79615                              ; Case 55 ('U') of switch 004AD06A
===85
   mov ebx,4D7899D                              ; Case 56 ('V') of switch 004AD06A
 mov ebx,4D82F05                              ; Case 57 ('W') of switch 004AD06A
    mov ebx,4267877                              ; Case 58 ('X') of switch 004AD06A
  mov ebx,4D47CD6                              ; Case 59 ('Y') of switch 004AD06A
    mov ebx,5026CBB                              ; Case 5A ('Z') of switch 004AD06A
   mov ebx,4D78B01                              ; Case 5B ('[') of switch 004AD06A
   mov ebx,4D78A4D                              ; Case 5C ('') of switch 004AD06A
  mov ebx,508853F                              ; Case 5D (']') of switch 004AD06A
  mov ebx,4205E5B                              ; Case 5E ('^') of switch 004AD06A
   mov ebx,4E6CB3B                              ; Case 5F ('_') of switch 004AD06A
  mov ebx,571A622                              ; Case 60 ('`') of switch 004AD06A
   mov ebx,5E72823                              ; Case 61 ('a') of switch 004AD06A
  mov ebx,3AAE61E                              ; Case 62 ('b') of switch 004AD06A
  mov ebx,5F0461F                              ; Case 63 ('c') of switch 004AD06A,
  mov ebx,3ADF8D6                              ; Case 64 ('d') of switch 004AD06A
  mov ebx,176D3C5                              ; Case 65 ('e') of switch 004AD06A
```



```asm
mov ebx,4D78AB9 ; Case 66 ('f') of switch 004AD06A
mov ebx,4F60F77 ; Case 67 ('g') of switch 004AD06A
mov ebx,4D78B5A ; Case 68 ('h') of switch 004AD06A
mov ebx,50A5AC5 ; Case 69 ('i') of switch 004AD06A
```

【Greetings】 看雪论坛, `FCG` 论坛, `DFCG` 论坛等

【完稿时间等】`2005.04.30`，下午 `05:06`,天气:多云,广州

`4dd466d0e1e0cc123afbe504da2462ca`
