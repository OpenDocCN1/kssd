
# SmartCheck 6.03

> 原文：[`www.pediy.com/kssd/pediy03/forum1248.htm`](https://www.pediy.com/kssd/pediy03/forum1248.htm)

SmartCheck 6.03 的 InstallShield 序列号破解。

Passion

2001.8.20

SmartCheck 6.03 对于破解者来说不但是对付 VB 的利器，而且它自个儿也算是典型的 InstallShield 安装的破解范例。Numega 的产品的检验过程似乎都类似。我以前不完全地破过 SoftICE，因为不全，感觉总不对，这次拿 SmartCheck 重新来一次，希望过程还算完整。

先拿 Windows instalshield decompiler 也就是 `wisdec` 反 `setup.ins`，反出来一大段。有过一点破 Sice 的经验，也不浪费时间查找字符串“The Serial Number……”了，反正找不到。看看它导入了什么 DLL 文件，也就是看 MISC 菜单下的 DLL import，有俩比较特别，是 `UTILITY.DLL` 和 `NMINST32.DLL`，其中 `UTILITY.DLL` 有个导出的函数叫 `DigitCheck`，一个参数还是 STRING，似乎专门就是检验序列号的，这么容易？

在反编译后的文本中找字串 `UTILITY.DLL`，很容易就找到一段：

```
StrLocal[0007] = SUPPORTDIR ^ "UTILITY.dll"

00004C96: 0128  IF (UseDLL (StrLocal[0007]) = 00000000) THEN
00004CB6: 00B4          NumLocal[0001] = UTILITY.DigitCheck (StrLocal[0002])
00004CC6: 00B3          UnUseDLL (StrLocal[0007])
00004CC7: 0000  ENDIF
```

——这是典型的动态加载 `UTILITY.DLL`，检验序列号再释放该 DLL 的过程。

动态加载好办，至少可以断 `LoadLibraryA`。

好，现在开始动态跟踪。启动 SmartCheck 的 `setup.exe`，下一步下一步，嗯？怎么序列号没填就通过了？——对了，SmartCheck 安装的时候会检验注册表里是否有现存的 NUmega 的同种产品，有的话就自动读出序列号来填到序列号框里去，真是体贴人哇。——咱们这群人谁没装 Numega 的东西？^_^

算了算了，按上一步回到序列号填写的界面。Ctrl＋D 切入 Sice，下断 `bpx loadlibrarya do "d esp->4"`

回来按下一步，断了，正是加载 `utility.dll`。

且慢，这是正确的序列号，倘若乱填的呢？——真的乱填一气，按下一步，糟！直接出来序列号错误的消息框了。

看来序列号有两步检验过程，第一步通过后才 `loadlibraryA` 来加载 `UTILITY.DLL` 来进行第二次检验。怎么办？

断 `messageboxa` 的话也有效，断后返回 `nminst32.dll` 的领空（在我的机子上 `call messageboxa` 的地址是 2491cf9，`mod nmi` 命令可看见 `NMINST32.DLL` 的基址是 2490000），只是只能跳过检验，不能帮我们弄出正确的序列号来。

看来是 `nminst32.dll` 在弄鬼，再静态的反反看，用 Win32Dasm 反一下 `nminst32.dll`，10001cf9 处果然是 `Call MessageBoxa`，往上看，一大段 REPNE SCANSB 加 MOVSB 的，也不知道在干什么。

往上面找个断点，既不能太远进不来，也不能太近（怕已经出错了）。——这段过程好像是用 `SendDlgItemmessagea` 来获取文本框内容并且进行第一步检验，对了就 `EndDialog`。前后找找看就知道了。

选个断点 1A70，也就是 2491a70，是输出函数的开始：

```
:10001A70 8B442408            mov eax, dword ptr [esp+08]
:10001A74 81EC64020000        sub esp, 00000264
:10001A7A 83E810              sub eax, 00000010
:10001A7D 53                  push ebx
:10001A7E 55                  push ebp
:10001A7F 56                  push esi
:10001A80 57                  push edi
:10001A81 0F8490050000        je 10002017
:10001A87 2D00010000          sub eax, 00000100
:10001A8C 0F841C040000        je 10001EAE
:10001A92 48                  dec eax
:10001A93 0F858E050000        jne 10002027
:10001A99 8B842480020000      mov eax, dword ptr [esp+00000280]
:10001AA0 25FFFF0000          and eax, 0000FFFF
:10001AA5 83F80C              cmp eax, 0000000C
:10001AA8 0F8F76030000        jg 10001E24
:10001AAE 0F8461030000        je 10001E15
:10001AB4 48                  dec eax
:10001AB5 7416                je 10001ACD
:10001AB7 48                  dec eax
:10001AB8 0F8569050000        jne 10002027
```

其实这个断点没啥效果，程序没事就进来（这段输出函数是对话框的过程，随便动点什么东西都会进来）。

不过跟了几次后觉得前面的是检查点击了什么按钮的，看看这儿 1AB5 处的跳

```
:10001AB5 7416                je 10001ACD
```

其他几个跳都很远（差不多就是直接退出对话框过程），而且这个跳指向的地方附近就有 `SendDlgItemMessagea` 函数引用。估计这里跳的话可能有文章，于是下断 2491acd

（不知怎的，setup 界面中直接 CTRL+D 切入再下断的话大多数不成功，不是没效果就是 Invalid address，难道填序列号的时候 `nminsl32.dll` 还没装进来？不管它，填满后先断 `hmemcpy` 进来，回 Nminst32 领空再来下断 `bpx 2491acd`，注意得清掉 `hmemcpy` 的断点。）

好，断了！F10 往下走:

```
* Reference To: USER32.SendDlgItemMessageA, Ord:01D8h
                                  |
:10001AD4 8B1D30230210            mov ebx, dword ptr [10022330]
 …………
:10001AEC FFD3                    call ebx
 …………
:10001B00 FFD3                    call ebx
 …………
:10001B11 FFD3                    call ebx
```

连续三次 Call `USER32.SendDlgItemMessageA`，目的八成就是获得三个序列号框中的内容。

```
:10001B1A 33C0                    xor eax, eax
:10001B1C F2                      repnz     //这里看看 EDI，就是输入的序列号的前四个字符。
:10001B1D AE                      scasb
```

跟下来一大段 REPNE MOVSB SCANSB 之类的，不容易弄明白什么功能，直到下面这里：

```
:10001C52 8D442410                lea eax, dword ptr [esp+10]
    //这里 EAX 看看，是序列号第一部分的前三个字符，于是可以明白，前面是取三个字符。^_^
:10001C56 33F6                    xor esi, esi
:10001C58 50                      push eax
:10001C59 68608F0110              push 10018F60
:10001C5E E8DD860000              call 1000A340     //对这前三个字符进行检验。
```

10001C5E 这里来个 `d *esp`，看见什么了？一大堆数字：640 641 480 481 490 491 510 511 520 521

跟进去看看，原来:10001C5E 处的这个 Call 是检验输入的序列号的前三位数字是不是这一堆中的，不是？不是当然就不行！

```
:10001C66 85C0                    test eax, eax
:10001C68 7542                    jne 10001CAC
    //属于这一堆的话，返回那个符合的地址，否则返回 0，不跳就完了。
```

稍微改改数据，接着跳：

```
:10001CAC 6820A60110              push 1001A620
:10001CB1 E85A810000              call 10009E10     //这里是第二步检验。
:10001CB6 83C404                  add esp, 00000004
:10001CB9 8BF0                    mov esi, eax
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10001CAA(C)

```
:10001CBB 85F6                    test esi, esi
:10001CBD 7552                    jne 10001D11      //这里不跳也完蛋。
```

现在，序列号判断的流程都清楚了。——如果从:10001CB1 处的 CALL 进去，里面会来个 `Loadlibrarya`，这才真正的装入 `utility.dll` 文件。接着 `GetProcAddress`，传参数

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10009E75(C)

```
:10009E80 6890A20110              push 1001A290        //含 utility.dll 路径的字串
* Reference To: KERNEL32.LoadLibraryA, Ord:0190h
                                  |
:10009E85 FF1518220210            Call dword ptr [10022218]   //装 utility.dll
:10009E8B 8BF0                    mov esi, eax
:10009E8D 85F6                    test esi, esi        //esi 返回 DLL 的句柄。
:10009E8F 7539                    jne 10009ECA         //成功则跳。
```

接着 `GetProcAddress`，传参数调用

* Possible StringData Ref from Data Obj ->"DigitCheck"

```
:10009ECA 68C0520110              push 100152C0
:10009ECF 56                      push esi
* Reference To: KERNEL32.GetProcAddress, Ord:0116h
                                  |
:10009ED0 FF151C220210            Call dword ptr [1002221C]   //取 DigitCheck 函数的地址
:10009ED6 85C0                    test eax, eax
:10009ED8 740E                    je 10009EE8
:10009EDA 8D4C240C                lea ecx, dword ptr [esp+0C]
    //ECX 指向我们输入的序列号，已经去掉了“-”号的，
:10009EDE 51                      push ecx
:10009EDF FFD0                    call eax           //调用，
:10009EE1 83C404                  add esp, 00000004
:10009EE4 8BD8                    mov ebx, eax
:10009EE6 EB2D                    jmp 10009F15
```

10009EDF 处以刚刚推入堆栈的序列号的地址为参数调用 `utility.DigitCheck`，是应该跟进去的时候了。

（接上）

对话框中填的序列号的格式为 0000-000000-00，当然数字也可以为字母。

前面的第一步判断，前三位必须是那一堆字符中的数字，因此我们前三个填 510，后面仍然乱填一气（这里填 5101-234567-FF），先跟进去再说。

下面比较零乱，是我以前分析 Softice 的 `utility.dll` 时留下的记录，SmartCheck 的 `utility.dll` 结构几乎和它一模一样，许多东西玩起来也就轻松了。^_^

Exported fn(): `DigitCheck` - Ord:0001h

```
:10001110 56                      push esi
:10001111 57                      push edi
:10001112 8B7C240C                mov edi, dword ptr [esp+0C]   //序列号地址
:10001116 83C9FF                  or ecx, FFFFFFFF
:10001119 33C0                    xor eax, eax
:1000111B F2                      repnz
:1000111C AE                      scasb
:1000111D F7D1                    not ecx         //求长度。
:1000111F 2BF9                    sub edi, ecx
:10001121 8BC1                    mov eax, ecx
:10001123 8BF7                    mov esi, edi
:10001125 BF80CB0010              mov edi, 1000CB80
:1000112A C1E902                  shr ecx, 02
:1000112D F3                      repz
:1000112E A5                      movsd
:1000112F 8BC8                    mov ecx, eax
:10001131 83E103                  and ecx, 00000003
:10001134 F3                      repz
:10001135 A4                      movsb             //搬到一个地方去。
```



:10001136 `E895000000`                call `100011D0`

:1000113B `E840010000`                call `10001280`

:10001140 `33C9`                      xor `ecx`, `ecx`

:10001142 `890DACCB0010`              mov dword ptr `[1000CBAC]`, `ecx`

//这两个 `CALL` 会根据序列号的前 8 位数字经过查表运算得来一系列值，也就内定了前八个字符必须是数字。详细内容见后面。

计算出来八个值：`04 0C 09 04 06 0A 0A 0B`，放在 `1000CBB0` 处。

:10001148 `8A81B0CB0010`              mov `al`, byte ptr `[ecx+1000CBB0]`    //取第一个

:1000114E `8A91B4CB0010`              mov `dl`, byte ptr `[ecx+1000CBB4]`    //取第四个

:10001154 `32C2`                      xor `al`, `dl`                  //先异或

//八个值中，第一个值和第四个值异或，再和最后一个值或，结果如不是数字或者字母则加 7 变成字母，比较后通过则继续计算第二个和第五个，并且仍和最后一个或，依此类推。

:10001156 `8A15B7CB0010`              mov `dl`, byte ptr `[1000CBB7]`         //一直都是取最后一个

:1000115C `0AC2`                      or `al`, `dl`                   //再或

:1000115E `0C30`                      or `al`, `30`

:10001160 `3C39`                      cmp `al`, `39`

:10001162 `8881B8CB0010`              mov byte ptr `[ecx+1000CBB8]`, `al`    //存结果

:10001168 `7E08`                      jle `10001172`

:1000116A `0407`                      add `al`, `07`                  //变字母

:1000116C `8881B8CB0010`              mov byte ptr `[ecx+1000CBB8]`, `al`    //也是存结果

//这里的 `AL` 是算出来的一个字符，算出一个后马上跟输入的序列号的最后四位比较。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001168(C)

|

:10001172 `8A81B8CB0010`              mov `al`, byte ptr `[ecx+1000CBB8]`

:10001178 `8A9188CB0010`              mov `dl`, byte ptr `[ecx+1000CB88]`

//这是输入序列号的后四位地址。前八位数字计算出来的四个字符必须和序列号末尾的四个字母一样才行。

:1000117E `3AC2`                      cmp `al`, `dl`

:10001180 `7406`                      je `10001188`                //比！

:10001182 `0C20`                      or `al`, `20`

:10001184 `3AC2`                      cmp `al`, `dl`

:10001186 `7514`                      jne `1000119C`               //不等就完蛋了

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001180(C)

|

:10001188 `41`                        inc `ecx`

:10001189 `83F904`                    cmp `ecx`, `00000004`         //比完四对就……

:1000118C `7CBA`                      jl `10001148`

:1000118E `890DACCB0010`              mov dword ptr `[1000CBAC]`, `ecx`

:10001194 `B801000000`                mov `eax`, `00000001`         //正确标志

:10001199 `5F`                        pop `edi`

:1000119A `5E`                        pop `esi`

:1000119B `C3`                        ret

这里程序算出来的四个字符是“`BFBF`”

——————————————————————————————

这里是第一个 `CALL` 的内容，它的功能也就是先根据输入的前 8 个字符查一张表，如果查出来的东西符合要求（也就是前 8 个字符都是数字）则把前 8 个字符的 `ASCII` 码都转换成实际的数字存放在 `1000CBB8` 处：

* Referenced by a `CALL` at Address:

|:10001136

|

:100011D0 `53`                        push `ebx`

:100011D1 `33D2`                      xor `edx`, `edx`

:100011D3 `33DB`                      xor `ebx`, `ebx`

:100011D5 `56`                        push `esi`

:100011D6 `8915A8CB0010`              mov dword ptr `[1000CBA8]`, `edx`

:100011DC `33C9`                      xor `ecx`, `ecx`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001240(U)

|

:100011DE `A18C930010`                mov `eax`, dword ptr `[1000938C]`

:100011E3 `890DACCB0010`              mov dword ptr `[1000CBAC]`, `ecx`

:100011E9 `83F801`                    cmp `eax`, `00000001`

:100011EC `7E20`                      jle `1000120E`

//也不知是什么判断，反正总是跳。

:100011EE `0FBE8180CB0010`            movsx `eax`, byte ptr `[ecx+1000CB80]`

:100011F5 `6A04`                      push `00000004`

:100011F7 `50`                        push `eax`

:100011F8 `E883080000`                call `10001A80`

:100011FD `8B15A8CB0010`              mov `edx`, dword ptr `[1000CBA8]`

:10001203 `8B0DACCB0010`              mov `ecx`, dword ptr `[1000CBAC]`

:10001209 `83C408`                    add `esp`, `00000008`

:1000120C `EB13`                      jmp `10001221`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:100011EC(C)

|

:1000120E `0FBE8180CB0010`            movsx `eax`, byte ptr `[ecx+1000CB80]`    //表首地址

* Possible StringData Ref from Data Obj ->"             (((((              "

                                          ->"  H 剟剟剟剟剟亖亖亖"

//第一张表的内容                    ->"倐倐倐"

                                          ->" "

                                          |

:10001215 `8B3580910010`              mov `esi`, dword ptr `[10009180]`

:1000121B `8A0446`                    mov `al`, byte ptr `[esi+2*eax]`

:1000121E `83E004`                    and `eax`, `00000004`

//下面说明前八个字符必须都是数字。后四个字符根据这些数字来计算。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1000120C(U)

|

:10001221 `3BC3`                      cmp `eax`, `ebx`

:10001223 `741D`                      je `10001242`          //如果输入的不是数字，则这里会跳。

:10001225 `83F908`                    cmp `ecx`, `00000008`    //满了 8 个则跳。

:10001228 `7D18`                      jge `10001242`

:1000122A `8A8180CB0010`              mov `al`, byte ptr `[ecx+1000CB80]`

:10001230 `240F`                      and `al`, `0F`

:10001232 `8882B0CB0010`              mov byte ptr `[edx+1000CBB0]`, `al`

:10001238 `42`                        inc `edx`

:10001239 `8915A8CB0010`              mov dword ptr `[1000CBA8]`, `edx`

:1000123F `41`                        inc `ecx`

:10001240 `EB9C`                      jmp `100011DE`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:10001223(C), :10001228(C)

|

:10001242 `83FA08`                    cmp `edx`, `00000008`

:10001245 `7D33`                      jge `1000127A`

:10001247 `4A`                        dec `edx`

:10001248 `B807000000`                mov `eax`, `00000007`

:1000124D `8915A8CB0010`              mov dword ptr `[1000CBA8]`, `edx`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1000126D(C)

|

:10001253 `3BD3`                      cmp `edx`, `ebx`

:10001255 `7D08`                      jge `1000125F`

:10001257 `8898B0CB0010`              mov byte ptr `[eax+1000CBB0]`, `bl`

:1000125D `EB0D`                      jmp `1000126C`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001255(C)

|

:1000125F `8A8AB0CB0010`              mov `cl`, byte ptr `[edx+1000CBB0]`

:10001265 `4A`                        dec `edx`

:10001266 `8888B0CB0010`              mov byte ptr `[eax+1000CBB0]`, `cl`

//这里 `1000cbb0` 处执行完后就存放了前 8 位数字的实际值，也就是

`05 01 00 01 02 03 04 05`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1000125D(U)

|

:1000126C `48`                        dec `eax`

:1000126D `79E4`                      jns `10001253`

:1000126F `A3ACCB0010`                mov dword ptr `[1000CBAC]`, `eax`

:10001274 `8915A8CB0010`              mov dword ptr `[1000CBA8]`, `edx`

//这里存放俩处理过的字节数，都是 8。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001245(C)

|

:1000127A `5E`                        pop `esi`

:1000127B `5B`                        pop `ebx`

:1000127C `C3`                        ret

下面是第二个 `CALL` 的内容，这个 `CALL` 把第一个 `CALL` 计算出来的数字再查一次表，结果覆盖了 `1000cbb0` 处的 8 个字节：

* Referenced by a `CALL` at Address:

|:1000113B

|

:10001280 `33C0`                      xor `eax`, `eax`

:10001282 `A3AC910010`                mov dword ptr `[100091AC]`, `eax`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:100012A7(C)

|

:10001287 `0FBE88B0910010`            movsx `ecx`, byte ptr `[eax+100091B0]`

:1000128E `8B148590800010`            mov `edx`, dword ptr `[4*eax+10008090]`    //又查一次表

:10001295 `40`                        inc `eax`

:10001296 `83F808`                    cmp `eax`, `00000008`               //处理 8 个字符

:10001299 `8A0C0A`                    mov `cl`, byte ptr `[edx+ecx]`

:1000129C `A3AC910010`                mov dword ptr `[100091AC]`, `eax`

:100012A1 `8888AF910010`              mov byte ptr `[eax+100091AF]`, `cl`    //存查表结果

:100012A7 `7CDE`                      jl `10001287`

:100012A9 `C3`                        ret                             //返回

查出来的结果仍然放在 `1000cbb0`，这里变成了 `04 0C 09 04 06 0A 0A 0B`

最后得可以用的序列号： `5101-2345BF-BF`

说明：此段和 `SoftICE` 的序列号破解过程除了指令定位地址不一样外，其他几乎一模一样，所以连说明文字都类似，`*_*`



至于注册机，应该也能写出来，但时间仓促，……。倘若没皮兄的“鞭策”，唉，…… 

```delphi
unit Unit1;

interface

uses
  Windows, Messages, SysUtils, Classes, Graphics, Controls, Forms, Dialogs,
  StdCtrls;

type
  TForm1 = class(TForm)
    Edit1: TEdit;
    Button1: TButton;
    Button2: TButton;
    Button3: TButton;
    procedure Button3Click(Sender: TObject);
    procedure Button1Click(Sender: TObject);
    procedure Button2Click(Sender: TObject);
  private
    { Private declarations }
  public
    { Public declarations }
  end;

var
  Form1: TForm1;

//这里是第二个表的内容。第一个表用条件判断省略了。

const table: array[1..8, 0..15] of byte = (
  (15, 1, 11, 3, 8, 4, 14, 7, 13, 0, 0, 0, 0, 0, 0, 0),
  (10, 12, 1, 8, 2, 0, 9, 15, 5, 11, 0, 0, 0, 0, 0, 0),
  (9, 5, 12, 2, 7, 6, 15, 4, 14, 10, 0, 0, 0, 0, 0, 0),
  (3, 4, 12, 11, 1, 10, 13, 8, 0, 14, 0, 0, 0, 0, 0, 0),
  (13, 1, 6, 11, 8, 10, 14, 4, 3, 12, 0, 0, 0, 0, 0, 0),
  (7, 11, 6, 10, 5, 9, 4, 8, 0, 3, 0, 0, 0, 0, 0, 0),
  (0, 13, 3, 15, 10, 8, 2, 12, 4, 6, 0, 0, 0, 0, 0, 0),
  (9, 5, 13, 1, 3, 11, 12, 4, 2, 8, 0, 0, 0, 0, 0, 0)
);

implementation

{$R *.DFM}

//FORM 内一个 EDIT 框，三个 BUTTON

procedure TForm1.Button3Click(Sender: TObject);

begin
  self.Close;
end;

procedure TForm1.Button1Click(Sender: TObject);

var
  strlst: TStringList;
  i, j: integer;
  s: string;
  t: array[1..8] of integer;
  ts: array[1..4] of byte;
  tc: array[1..4] of char;
begin
  strlst := TStringList.Create;
  strlst.Add('640');
  strlst.Add('641');
  strlst.Add('480');
  strlst.Add('481');
  strlst.Add('490');
  strlst.Add('491');
  strlst.Add('510');
  strlst.Add('511');
  strlst.Add('520');
  strlst.Add('521');

  i := (GetTickCount mod 10) + 1;
  randomize;
  j := random(100000);

  s := inttostr(strtoint(strlst.strings[i]) * 100000 + j);

  for i := 1 to 8 do
  begin
    t[i] := table[i, strtoint(s[i])];
  end;

  for i := 1 to 4 do
  begin
    ts[i] := ((t[i] xor t[i + 4]) or t[8]) or $30;
    if (ts[i] > $39) then
      ts[i] := ts[i] + 7;
    tc[i] := chr(ts[i]);
  end;

  self.Edit1.Text := copy(s, 1, 4) + '-' + copy(s, 5, 4) + copy(tc, 1, 2) + '-' + copy(tc, 3, 2);
  strlst.Free;
end;

procedure TForm1.Button2Click(Sender: TObject);

begin
  messagebox(self.handle, 'SmartCheck 6.03 注册机' +#13#10 + '编写：Passion' +#13#10 + '2001 年 8 月 22 日', '关于', MB_OK);
end;

end.
```

`4dd466d0e1e0cc123afbe504da2462ca`
