
# 天音怒放 3.0

> 原文：[`www.pediy.com/kssd/pediy04/kanxue489.htm`](https://www.pediy.com/kssd/pediy04/kanxue489.htm)

作者：`Crack007`

主页：`http://www.crack007.com` (^-^)

破解日期：`2002-3-16`

破解工具：`TRW2000 1.22` `W32dASM 8.93` `Dede 3.02` 简体中文版 `fi2.49a` `Caspr1.10`

难度：★★

软件下载地址：`http://hn.skycn.net/down/typlayer.zip`

软件简介：

天音怒放是一款全功能的多媒体和 `MP3` 播放器。支持 `MP3, CD, WAV, MIDI, AVI, MPEG Video, Windows Media, QuickTime`，`MPEG4` 等格式，有众多令人心动的功能：歌词库能保存您喜爱的歌词, `CD` 转录器将您喜欢的歌从 `CD` 转换为 `MP3`，10 频段均衡器增强音乐的音色，自动保存播放清单。

主程序由 `fi` 侦测出是用 `ASPack v2.11` 加的壳。这个 `Esay`，用 `Caspr1.10` 轻松搞定。在此不得不额外在多嘴一句，那个 `ajj[CCG]` 编写的 `CasprGui` 很好使。我对其是怎么编写的十分有兴趣。因为我也想编写一些有关加参数 `DOS` 程序的外壳 (`Shell/GUI`)。希望大家尤其是 `ajj[CCG]` 大哥能给我提供一些资料。`OK`，言归正转。让我们继续聊聊这个软件的加密过程。

由于他是采用 `Delphi` 编写的软件，因此 `Dede` 此时就派上了大用场。在对付 `Delphi` 程序时，`DeDe` 相对 `W32dASM` 而言有着非常大的优势。其中最有用的莫过于他能分析出程序所采用的具体的控件和相关 `delphi` 函数。这样在遇到某些 `Call` 的时候我们可以非常快的判断出某段代码有什么作用，避免了钻 `Call` 的牛角尖，节省了大量的时间和精力。这在后面我们将会看到。

经过 `Dede` 对脱壳文件的处理之后，在窗体部分我们会看到一个 `TRegisterForm` 的模块，内容如下：

```
object RegisterForm: TRegisterForm
......
object Label2: TLabel
Left = 20
Top = 72
Width = 31
Height = 13
Caption = 'Name:'                <----姓名编辑框
Transparent = True
end
object Label3: TLabel
Left = 20
Top = 100
Width = 28
Height = 13
Caption = 'Code:'                <----注册码编辑框
Transparent = True
end
object bnRegister: TButton
Left = 176
Top = 132
Width = 65
Height = 25
Caption = '&OK'                 <----确认按钮
ModalResult = 1
TabOrder = 3
OnClick = bnRegisterClick        <----非常重要，程序就是从这个动作(OnClick)开始注册过程！
end                              让我们记下它的名称：bnRegisterClick
.....
```

让我们再来到程序一栏，找到 `TRegisterForm`，单击，在右边的时间栏里我们就会看到 `bnRegisterClick` 这个事件。双击可从此 `RVA` 处进行反汇编，得到内容如下：

```
***** TRY
|
004E708A 64FF30                 push    dword ptr fs:[eax]
004E708D 648920                 mov     fs:[eax], esp
004E7090 B201                   mov     dl, $01
* Reference to class TProcessRegister                              <----开始注册流程
|
004E7092 A104A44E00             mov     eax, dword ptr [$4EA404]
* Reference to: system.TObject.Create(TObject;Boolean);
|
004E7097 E820BFF1FF             call    00402FBC
004E709C 8BD8                   mov     ebx, eax
004E709E 8BC3                   mov     eax, ebx
|
004E70A0 E80F770000             call    004EE7B4
004E70A5 8D55F8                 lea     edx, [ebp-$08]
* Reference to control TRegisterForm.edUserName : TEdit              <----从 UserName : TEdit 里获取用户名
|
004E70A8 8B86EC020000           mov     eax, [esi+$02EC]
* Reference to: controls.TControl.GetText(TControl):System.String;
|
004E70AE E80DF8F4FF             call    004368C0
004E70B3 8B55F8                 mov     edx, [ebp-$08]
* Reference to field TProcessRegister.OFFS_0154
|
004E70B6 8D8354010000           lea     eax, [ebx+$0154]
* Reference to: system.@LStrAsg;
|
004E70BC E863CCF1FF             call    00403D24
004E70C1 8D55F4                 lea     edx, [ebp-$0C]
* Reference to control TRegisterForm.edRegCode : TEdit               <----从 RegCode : TEdit 里获取注册码
|
004E70C4 8B86F0020000           mov     eax, [esi+$02F0]
* Reference to: controls.TControl.GetText(TControl):System.String;    // 这就是 DeDe 的好处，一目了然，
|                                                                             再往后你会体会得更深刻！ *^_*
004E70CA E8F1F7F4FF             call    004368C0
004E70CF 8B55F4                 mov     edx, [ebp-$0C]
004E70D2 8BC3                   mov     eax, ebx
|
004E70D4 E8EB760000             call    004EE7C4                        <----在此行双击进入此 Call
* Reference to field TProcessRegister.OFFS_015C
|
004E70D9 80BB5C01000000         cmp     byte ptr [ebx+$015C], $00       <---很经典的比对，不是么？
004E70E0 0F84C8000000           jz      004E71AE
004E70E6 6A00                   push    $00
004E70E8 668B0D10724E00         mov     cx, word ptr [$4E7210]
004E70EF B202                   mov     dl, $02
* Possible String Reference to: '谢谢你选择了天音怒放'
|
004E70F1 B81C724E00             mov     eax, $004E721C
|
004E70F6 E8496AF7FF             call    0045DB44
004E70FB 8D55F0                 lea     edx, [ebp-$10]
----------------------------------call    004EE7C4---------------------------------------------
***** TRY
|
004EE7F9 64FF30                 push    dword ptr fs:[eax]
004EE7FC 648920                 mov     fs:[eax], esp
* Reference to field TProcessRegister.OFFS_0158
|
004EE7FF 8D8658010000           lea     eax, [esi+$0158]
004EE805 8BD3                   mov     edx, ebx
* Reference to: system.@LStrAsg;
|
004EE807 E81855F1FF             call    00403D24
004EE80C 8BC3                   mov     eax, ebx
* Reference to: system.@LStrLen:Integer;                         <---- LStrLen:Integer，顾名思义，就是取子串的长度，
                                                                                   返回一个整数值（保存在 EAX）。也就说下面的那个
                                                                                   Call 你不用跟进去就知道是干什么的。方便吧！
004EE80E E83D57F1FF             call    00403F50
004EE813 83F80E                 cmp     eax, +$0E
004EE816 0F8C0C040000           jl      004EEC28                  <----这两句要求注册码的位数大于 14(=0Eh)位
004EE81C 8D45C4                 lea     eax, [ebp-$3C]
004EE81F 8A13                   mov     dl, byte ptr [ebx]
004EE821 885001                 mov     [eax+$01], dl
004EE824 C60001                 mov     byte ptr [eax], $01
004EE827 8D55C4                 lea     edx, [ebp-$3C]
004EE82A 8D45C0                 lea     eax, [ebp-$40]
* Reference to: system.@PStrCpy;                                  <---- PStrCpy 复制字串
|
004EE82D E8F242F1FF             call    00402B24
004EE832 8D45BC                 lea     eax, [ebp-$44]
004EE835 8A5301                 mov     dl, byte ptr [ebx+$01]
004EE838 885001                 mov     [eax+$01], dl
004EE83B C60001                 mov     byte ptr [eax], $01
004EE83E 8D55BC                 lea     edx, [ebp-$44]
004EE841 8D45C0                 lea     eax, [ebp-$40]
004EE844 B102                   mov     cl, $02                   <---- 注意赋给 cl 的规律
* Reference to: system.@PStrNCat;                                  <---- PStrNCat 字串合并
|
004EE846 E8A942F1FF             call    00402AF4
004EE84B 8D55C0                 lea     edx, [ebp-$40]
004EE84E 8D45B8                 lea     eax, [ebp-$48]
* Reference to: system.@PStrCpy;
|
004EE851 E8CE42F1FF             call    00402B24
004EE856 8D45BC                 lea     eax, [ebp-$44]
004EE859 8A5302                 mov     dl, byte ptr [ebx+$02]
004EE85C 885001                 mov     [eax+$01], dl
004EE85F C60001                 mov     byte ptr [eax], $01
004EE862 8D55BC                 lea     edx, [ebp-$44]
004EE865 8D45B8                 lea     eax, [ebp-$48]
004EE868 B103                   mov     cl, $03                   <---- 取 3 个字符
* Reference to: system.@PStrNCat;
|
004EE86A E88542F1FF             call    00402AF4
004EE86F 8D55B8                 lea     edx, [ebp-$48]
004EE872 8D45F0                 lea     eax, [ebp-$10]
* Reference to: system.@LStrFromString(String;ShortString);       <---- 获取子串
|
004EE875 E87A56F1FF             call    00403EF4
004EE87A 8D45C4                 lea     eax, [ebp-$3C]
004EE87D 8A5304                 mov     dl, byte ptr [ebx+$04]
004EE880 885001                 mov     [eax+$01], dl
```



```
004EE883  C60001               mov    byte ptr [eax], $01   

004EE886  8D55C4               lea    edx, [ebp-$3C]   

004EE889  8D45C0               lea    eax, [ebp-$40]   

* Reference to: `system.@PStrCpy`;   

|   

004EE88C  E89342F1FF           call   `00402B24`   

004EE891  8D45BC               lea    eax, [ebp-$44]   

004EE894  8A5305               mov    dl, byte ptr [ebx+$05]   

004EE897  885001               mov    [eax+$01], dl   

004EE89A  C60001               mov    byte ptr [eax], $01   

004EE89D  8D55BC               lea    edx, [ebp-$44]   

004EE8A0  8D45C0               lea    eax, [ebp-$40]   

004EE8A3  B102                 mov    cl, $02                <---- 注意赋给 cl 的规律   

* Reference to: `system.@PStrNCat`;   

|   

004EE8A5  E84A42F1FF           call   `00402AF4`   

004EE8AA  8D55C0               lea    edx, [ebp-$40]   

004EE8AD  8D45B8               lea    eax, [ebp-$48]   

* Reference to: `system.@PStrCpy`;   

|   

004EE8B0  E86F42F1FF           call   `00402B24`   

004EE8B5  8D45BC               lea    eax, [ebp-$44]   

004EE8B8  8A5306               mov    dl, byte ptr [ebx+$06]   

004EE8BB  885001               mov    [eax+$01], dl   

004EE8BE  C60001               mov    byte ptr [eax], $01   

004EE8C1  8D55BC               lea    edx, [ebp-$44]   

004EE8C4  8D45B8               lea    eax, [ebp-$48]   

004EE8C7  B103                 mov    cl, $03   

* Reference to: `system.@PStrNCat`;   

|   

004EE8C9  E82642F1FF           call   `00402AF4`   

004EE8CE  8D55B8               lea    edx, [ebp-$48]   

004EE8D1  8D45B0               lea    eax, [ebp-$50]   

* Reference to: `system.@PStrCpy`;   

|   

004EE8D4  E84B42F1FF           call   `00402B24`   

004EE8D9  8D45BC               lea    eax, [ebp-$44]   

004EE8DC  8A5307               mov    dl, byte ptr [ebx+$07]   

004EE8DF  885001               mov    [eax+$01], dl   

004EE8E2  C60001               mov    byte ptr [eax], $01   

004EE8E5  8D55BC               lea    edx, [ebp-$44]   

004EE8E8  8D45B0               lea    eax, [ebp-$50]   

004EE8EB  B104                 mov    cl, $04   

* Reference to: `system.@PStrNCat`;   

|   

004EE8ED  E80242F1FF           call   `00402AF4`   

004EE8F2  8D55B0               lea    edx, [ebp-$50]   

004EE8F5  8D45A8               lea    eax, [ebp-$58]   

* Reference to: `system.@PStrCpy`;   

|   

004EE8F8  E82742F1FF           call   `00402B24`   

004EE8FD  8D45BC               lea    eax, [ebp-$44]   

004EE900  8A5308               mov    dl, byte ptr [ebx+$08]   

004EE903  885001               mov    [eax+$01], dl   

004EE906  C60001               mov    byte ptr [eax], $01   

004EE909  8D55BC               lea    edx, [ebp-$44]   

004EE90C  8D45A8               lea    eax, [ebp-$58]   

004EE90F  B105                 mov    cl, $05                <---- 取 5 个字符   

* Reference to: `system.@PStrNCat`;   

|   

004EE911  E8DE41F1FF           call   `00402AF4`   

004EE916  8D55A8               lea    edx, [ebp-$58]   

004EE919  8D45EC               lea    eax, [ebp-$14]   

* Reference to: `system.@LStrFromString(String;ShortString)`;   

|   

004EE91C  E8D355F1FF           call   `00403EF4`   

004EE921  8D45C4               lea    eax, [ebp-$3C]   

004EE924  8A530A               mov    dl, byte ptr [ebx+$0A]   

004EE927  885001               mov    [eax+$01], dl   

004EE92A  C60001               mov    byte ptr [eax], $01   

004EE92D  8D55C4               lea    edx, [ebp-$3C]   

004EE930  8D45C0               lea    eax, [ebp-$40]   

* Reference to: `system.@PStrCpy`;   

|   

004EE933  E8EC41F1FF           call   `00402B24`   

004EE938  8D45BC               lea    eax, [ebp-$44]   

004EE93B  8A530B               mov    dl, byte ptr [ebx+$0B]   

004EE93E  885001               mov    [eax+$01], dl   

004EE941  C60001               mov    byte ptr [eax], $01   

004EE944  8D55BC               lea    edx, [ebp-$44]   

004EE947  8D45C0               lea    eax, [ebp-$40]   

004EE94A  B102                 mov    cl, $02                <---- 注意赋给 cl 的规律   

* Reference to: `system.@PStrNCat`;   

|   

004EE94C  E8A341F1FF           call   `00402AF4`   

004EE951  8D55C0               lea    edx, [ebp-$40]   

004EE954  8D45B8               lea    eax, [ebp-$48]   

* Reference to: `system.@PStrCpy`;   

|   

004EE957  E8C841F1FF           call   `00402B24`   

004EE95C  8D45BC               lea    eax, [ebp-$44]   

004EE95F  8A530C               mov    dl, byte ptr [ebx+$0C]   

004EE962  885001               mov    [eax+$01], dl   

004EE965  C60001               mov    byte ptr [eax], $01   

004EE968  8D55BC               lea    edx, [ebp-$44]   

004EE96B  8D45B8               lea    eax, [ebp-$48]   

004EE96E  B103                 mov    cl, $03                


* Reference to: `system.@PStrNCat`;   

|   

004EE970  E87F41F1FF           call   `00402AF4`   

004EE975  8D55B8               lea    edx, [ebp-$48]   

004EE978  8D45B0               lea    eax, [ebp-$50]   

* Reference to: `system.@PStrCpy`;   

|   

004EE97B  E8A441F1FF           call   `00402B24`   

004EE980  8D45BC               lea    eax, [ebp-$44]   

004EE983  8A530D               mov    dl, byte ptr [ebx+$0D]   

004EE986  885001               mov    [eax+$01], dl   

004EE989  C60001               mov    byte ptr [eax], $01   

004EE98C  8D55BC               lea    edx, [ebp-$44]   

004EE98F  8D45B0               lea    eax, [ebp-$50]   

004EE992  B104                 mov    cl, $04                <---- 取 4 个字符   

* Reference to: `system.@PStrNCat`;   

|   

004EE994  E85B41F1FF           call   `00402AF4`   

004EE999  8D55B0               lea    edx, [ebp-$50]   

004EE99C  8D45E8               lea    eax, [ebp-$18]   

* Reference to: `system.@LStrFromString(String;ShortString)`;      <---- 根据上下文我们可以依稀认为注册码的形式为   

|                                                                 xxx-xxxxx-xxxx-xxx   

004EE99F  E85055F1FF           call   `00403EF4`                          ^^^^这部分可能有也可能没有   

004EE9A4  33D2                 xor    edx, edx   

004EE9A6  8B45F0               mov    eax, [ebp-$10]   

* Reference to: `sysutils.StrToIntDef(System.AnsiString;System.Integer):System.Integer`;  <----字符串转数字   

|   

004EE9A9  E8FAA9F1FF           call   `004093A8`   

004EE9AE  8945FC               mov    [ebp-$04], eax   

004EE9B1  33D2                 xor    edx, edx   

004EE9B3  8B45EC               mov    eax, [ebp-$14]   

* Reference to: `sysutils.StrToIntDef(System.AnsiString;System.Integer):System.Integer`;  <----字符串转数字   

|   

004EE9B6  E8EDA9F1FF           call   `004093A8`   

004EE9BB  8945F8               mov    [ebp-$08], eax   

004EE9BE  33D2                 xor    edx, edx   

004EE9C0  8B45E8               mov    eax, [ebp-$18]   

* Reference to: `sysutils.StrToIntDef(System.AnsiString;System.Integer):System.Integer`;  <----字符串转数字   

|   

004EE9C3  E8E0A9F1FF           call   `004093A8`   

004EE9C8  8BD8                 mov    ebx, eax   

004EE9CA  837DFC00             cmp    dword ptr [ebp-$04], +$00          <---- 下面这几行要求注册码不能为空   

004EE9CE  0F8454020000         jz     `004EEC28`   

004EE9D4  837DF800             cmp    dword ptr [ebp-$08], +$00   

004EE9D8  0F844A020000         jz     `004EEC28`   

004EE9DE  85DB                 test   ebx, ebx   

004EE9E0  0F8442020000         jz     `004EEC28`                          <---- 跳则死！   

|   

004EE9E6  E83589F1FF           call   `00407320`                          ---|  这里是一个小暗桩。主要作用是判断   

004EE9EB  2B8650010000         sub    eax, dword ptr [esi+$0150]             |\_\_ 前后两次执行设计代码的时间间隔。   
```



以下汇编代码段展示了程序中的反跟踪逻辑及注册码校验过程。主要包含时间间隔检查、余数运算验证、对象实例化及注册名哈希计算。

```
004EE9F1  3DF4010000            cmp     eax, $000001F4                      | 如果间隔过长（大于 500 ms），则认   
004EE9F6  0F872C020000          jnbe    004EEC28                           ---| 为程序被跟踪。注册失败。   
004EE9FC  8B45F8                mov     eax, [ebp-$08]                     ---|   
004EE9FF  0345FC                add     eax, [ebp-$04]                     |-- xxxxx + xxx + 8BF   
004EEA02  05F88B0000            add     eax, +$00008BF8                    ---|   
004EEA07  B910270000            mov     ecx, $00002710                     <---- 2170h = 10000d                     
004EEA0C  99                      cdq   
004EEA0D  F7F9                    idiv    ecx                              <---- 求余，edx 保存余数   
004EEA0F  8BCA                    mov     ecx, edx                         <---- 保存余数到 ecx   
004EEA11  8BF9                    mov     edi, ecx                         <---- 保存余数到 edi   
004EEA13  81E101000080            and     ecx, $80000001                   <---- 将余数和 80000001 做与运算，ecx 保存结果   
004EEA19  7905                    jns     004EEA20   
004EEA1B  49                      dec     ecx   
004EEA1C  83C9FE                  or      ecx, -$02   
004EEA1F  41                      inc     ecx   
004EEA20  2BF9                    sub     edi, ecx                         ---|   
004EEA22  8BC3                    mov     eax, ebx                         |\_\_ 要求注册码符合以下条件：   
004EEA24  03C0                    add     eax, eax                         |  余数 - ecx = 2 * xxxx   
004EEA26  3BF8                    cmp     edi, eax                         ---|   
004EEA28  0F85FA010000            jnz     004EEC28   

004EEA2E  B201                    mov     dl, $01   

* Reference to class TBitmap   

|   

004EEA30  A1D8C54100              mov     eax, dword ptr [$41C5D8]   

|   

004EEA35  E8EE46F3FF              call    00423128   

004EEA3A  8945E4                  mov     [ebp-$1C], eax   

* Possible String Reference to: 'CDROM'   

|   

004EEA3D  B974EC4E00              mov     ecx, $004EEC74   

004EEA42  8B15DC444F00            mov     edx, [$4F44DC]   

004EEA48  8B45E4                  mov     eax, [ebp-$1C]   

* Reference to: graphics.TBitmap.LoadFromResourceName(TBitmap;Windows.THandle;System.AnsiString);   

|   

004EEA4B  E89851F3FF              call    00423BE8   

004EEA50  B201                    mov     dl, $01   

* Reference to class TMemoryStream   

|   

004EEA52  A18C074100              mov     eax, dword ptr [$41078C]   

* Reference to: system.TObject.Create(TObject;Boolean);   

|   

004EEA57  E86045F1FF              call    00402FBC   

004EEA5C  8BD8                    mov     ebx, eax   

004EEA5E  8BD3                    mov     edx, ebx   

004EEA60  8B45E4                  mov     eax, [ebp-$1C]   

004EEA63  8B08                    mov     ecx, [eax]   

* Possible reference to virtual method TBitmap.OFFS_54   

|   

004EEA65  FF5154                  call    dword ptr [ecx+$54]   

004EEA68  33C9                    xor     ecx, ecx   

004EEA6A  BA40000000              mov     edx, $00000040   

004EEA6F  8BC3                    mov     eax, ebx   

004EEA71  8B38                    mov     edi, [eax]   

* Reference to method TMemoryStream.Seek(Longint,Word)   

|   

004EEA73  FF570C                  call    dword ptr [edi+$0C]   

* Reference to field TProcessRegister.OFFS_0160   

|   

004EEA76  8D9660010000            lea     edx, [esi+$0160]   

004EEA7C  B930000000              mov     ecx, $00000030   

004EEA81  8BC3                    mov     eax, ebx   

004EEA83  8B38                    mov     edi, [eax]   

* Reference to method TMemoryStream.Read(Pointer,Longint)   

|   

004EEA85  FF5704                  call    dword ptr [edi+$04]   

004EEA88  8B45E4                  mov     eax, [ebp-$1C]   

* Reference to: system.TObject.Free(TObject);   

|   

004EEA8B  E85C45F1FF              call    00402FEC   

004EEA90  8BC3                    mov     eax, ebx   

* Reference to: system.TObject.Free(TObject);   

|   

004EEA92  E85545F1FF              call    00402FEC   

|   

004EEA97  E88488F1FF              call    00407320   

004EEA9C  2B8650010000            sub     eax, dword ptr [esi+$0150]   

004EEAA2  3DF4010000              cmp     eax, $000001F4                   <---- 不能跳！原理同上。   
004EEAA7  0F877B010000            jnbe    004EEC28   

004EEAAD  C745D800000000          mov     dword ptr [ebp-$28], $00000000   

004EEAB4  C745DC00000000          mov     dword ptr [ebp-$24], $00000000   

004EEABB  8D55D4                  lea     edx, [ebp-$2C]   

* Reference to field TProcessRegister.OFFS_0154   

|   

004EEABE  8B8654010000            mov     eax, [esi+$0154]                 <---- 获得注册名   

* Reference to: sysutils.UpperCase(System.AnsiString):System.AnsiString;  <---- UpperCase 字符小写变大写   

|   

004EEAC4  E80BA4F1FF              call    00408ED4   

004EEAC9  8B45D4                  mov     eax, [ebp-$2C]   

* Reference to: system.@LStrLen:Integer;                                   <---- LStrLen:Integer 获取注册名长度   

|   

004EEACC  E87F54F1FF              call    00403F50   

004EEAD1  8BD8                    mov     ebx, eax   

004EEAD3  4B                      dec     ebx   

004EEAD4  85DB                    test    ebx, ebx                         <---- 要求注册名长度不能为 0   
004EEAD6  7C2F                    jl      004EEB07   

004EEAD8  43                      inc     ebx   

004EEAD9  33C9                    xor     ecx, ecx   

004EEADB  8BC1                    mov     eax, ecx                         ----|   
004EEADD  BF0C000000              mov     edi, $0000000C                   
004EEAE2  99                      cdq                                      
004EEAE3  F7FF                    idiv    edi                              |    这段循环的主要作用是：   
004EEAE5  8B849660010000          mov     eax, [esi+edx*4+$0160]           |   
004EEAEC  8B55D4                  mov     edx, [ebp-$2C]                  |    在 esi+$0160 处有一个码表，每 4   
004EEAEF  0FB6140A                movzx   edx, byte ptr [edx+ecx]         |   
004EEAF3  F7EA                    imul    edx                              |---  个字节为一单位，共有 12 值。   
004EEAF5  33D2                    xor     edx, edx                         |   
004EEAF7  0345D8                  add     eax, [ebp-$28]                  |    依次取注册名的 ASCII 与之相乘，   
004EEAFA  1355DC                  adc     edx, [ebp-$24]                  |   
004EEAFD  8945D8                  mov     [ebp-$28], eax                 |    并将结果进行累加   
004EEB00  8955DC                  mov     [ebp-$24], edx                 
004EEB03  41                      inc     ecx                              
004EEB04  4B                      dec     ebx                              
004EEB05  75D4                    jnz     004EEADB                       ----|   

004EEB07  B8FFFFFFFF              mov     eax, $FFFFFFFF   

004EEB0C  BAFFFFFF7F              mov     edx, $7FFFFFFF   

004EEB11  52                      push    edx   

004EEB12  50                      push    eax   

004EEB13  6A00                    push    $00   

004EEB15  6836010000              push    $00000136   

004EEB1A  8B45D8                  mov     eax, [ebp-$28]   

004EEB1D  8B55DC                  mov     edx, [ebp-$24]   

|   

004EEB20  E82781F1FF              call    00406C4C                       <--- 将结果乘以 0136h   

004EEB25  290424                  sub     dword ptr [esp], eax   

004EEB28  19542404                sbb     [esp+$04], edx                 7FFFFFFFh - 相乘后的结果   

004EEB2C  58                      pop     eax   

004EEB2D  5A                      pop     edx   

004EEB2E  52                      push    edx   

004EEB2F  50                      push    eax   

004EEB30  8D45F4                  lea     eax, [ebp-$0C]   

* Reference to: sysutils.IntToStr(System.Int64):System.AnsiString;overload;  <---- 整数转字符串
```



## 004EEB33 处的反汇编代码段

```text
004EEB33 E8BCA7F1FF call 004092F4
004EEB38 8D45D0 lea eax, [ebp-$30]
004EEB3B 50 push eax
004EEB3C B903000000 mov ecx, $00000003
004EEB41 BA07000000 mov edx, $00000007 <---- 从第 7 位开始取，共取 3 个
004EEB46 8B45F4 mov eax, [ebp-$0C] 假设为 yyy

* Reference to: system.@LStrCopy;

004EEB49 E80A56F1FF call 00404158
004EEB4E 8D45CC lea eax, [ebp-$34]
004EEB51 50 push eax
004EEB52 B905000000 mov ecx, $00000005
004EEB57 BA0A000000 mov edx, $0000000A <---- 从第 A 位开始取，共取 5 个
004EEB5C 8B45F4 mov eax, [ebp-$0C] 假设为 yyyyy

* Reference to: system.@LStrCopy;

004EEB5F E8F455F1FF call 00404158
004EEB64 8D45C8 lea eax, [ebp-$38]
004EEB67 50 push eax
004EEB68 B902000000 mov ecx, $00000002
004EEB6D BA0F000000 mov edx, $0000000F <---- 从第 F 位开始取，共取 2 个
004EEB72 8B45F4 mov eax, [ebp-$0C] 假设为 yy

* Reference to: system.@LStrCopy;

004EEB75 E8DE55F1FF call 00404158
004EEB7A 8B45D0 mov eax, [ebp-$30]

* Reference to: sysutils.StrToInt(System.AnsiString):System.Integer; <---- 字符转数字

004EEB7D E8A6A7F1FF call 00409328
004EEB82 8BD8 mov ebx, eax
004EEB84 8B45CC mov eax, [ebp-$34]

* Reference to: sysutils.StrToInt(System.AnsiString):System.Integer; <---- 字符转数字

004EEB87 E89CA7F1FF call 00409328
004EEB8C 8BF8 mov edi, eax
004EEB8E 8B45C8 mov eax, [ebp-$38]

* Reference to: sysutils.StrToInt(System.AnsiString):System.Integer; <---- 字符转数字

004EEB91 E892A7F1FF call 00409328
004EEB96 8B55F8 mov edx, [ebp-$08] ---|
004EEB99 33D0 xor edx, eax | 此处要求注册码符合以下条件：
004EEB9B 3BFA cmp edi, edx |
004EEB9D 0F8585000000 jnz 004EEC28 |-- yy xor yyyyy = xxxxx
004EEBA3 3345FC xor eax, [ebp-$04] |
004EEBA6 3BD8 cmp ebx, eax | yy xor yyy = xxx
004EEBA8 757E jnz 004EEC28 ---|

* Reference to field TProcessRegister.OFFS_015C

004EEBAA C6865C01000001 mov byte ptr [esi+$015C], $01
004EEBB1 8D45A4 lea eax, [ebp-$5C]

* Possible String Reference to: 'Player.ini'

004EEBB4 B984EC4E00 mov ecx, $004EEC84
004EEBB9 8B1580D44F00 mov edx, [$4FD480]
......
****** END
```

到这里，该注册码的前三部分就已经分析完毕了。大概步骤就是由注册名得出前两部分的注册码 `yyy-yyyyy`（见 `004EEAD9` 和 `004EEB96`），然后由后两部分再推出第三部分注册码 `yyy`（见 `004EEA0D`）。

那么是否到此就基本结束了呢？我们可以具体验证一下。填入注册名 `Crack007`，注册码 `887-32717-4718`，显示“谢谢你选择了天音怒放”。

好像是没多大问题了。但关闭后再运行软件，依旧显示未注册版本。怎么回事呢？让我们仔细想一想，问题究竟出在哪里。

在假定前面分析正确的基础上我们设想，如果软件在启动时没有在其他地方进行比对，而是单单进入刚才我们所分析的地方，那么应该是没有什么问题的。但现在出了问题，就说明软件在启动时还在其他地方进行了注册码的比对。那么到底是在哪里呢？

我们很容易得知软件的注册信息保存在其安装目录下的 `Player.ini` 文件当中。具体格式为：

```ini
[Register]
Name=Crack007
Code=887-32717-4718
```

因此我们用 `w32dasm` 对其反汇编后在字符串参考里面找 `Name`。为什么不选用 `player.ini` 做搜索的关键词呢？因为 `player.ini` 还包含有其他信息，调用的地方比较多，不利于我们的跟踪。因此我们最好选用 `Name` 或 `Code` 来查看。

在字符串参考里面我们不断双击 `Name` 字串，可以发现共有 4 个地方调用到字串 `Name`。其中 3 个都在刚才我们分析的代码范围之内，只有一个例外。看来问题多半就出在他身上。让我们到那里仔细看看：

```text
-------------------------------------------------------------------------------------
* Referenced by a CALL at Address:
|:004E7BD3 <---- 原来是在 004E7BD3 转过来的。到 004E7BD3 那边看看去。
|
:004EE41C 55 push ebp
:004EE41D 8BEC mov ebp, esp
:004EE41F 6A00 push 00000000
:004EE421 6A00 push 00000000
:004EE423 6A00 push 00000000

......

:004EE455 E89A4CF9FF call 004830F4
:004EE45A 8BF0 mov esi, eax
:004EE45C 8BC3 mov eax, ebx
:004EE45E E851030000 call 004EE7B4
:004EE463 6A00 push 00000000
:004EE465 8D45F8 lea eax, dword ptr [ebp-08]
:004EE468 50 push eax

* Possible StringData Ref from Code Obj ->"Name" <---- 到此处，向上看
|
:004EE469 B9F4E44E00 mov ecx, 004EE4F4

------------------------------------------------------------------------------------

***** TRY

......

004E7BD3 E844680000 call 004EE41C <---- 来源于这里，

* Reference to field TProcessRegister.OFFS_015C

004E7BD8 80BE5C01000000 cmp byte ptr [esi+$015C], $00
004E7BDF 7407 jz 004E7BE8 <---- 判断注册码是否符合前面的条件
004E7BE1 8BC6 mov eax, esi

004E7BE3 E838690000 call 004EE520 <---- 关键 Call，用 Dede 双击跟进

* Reference to field TProcessRegister.OFFS_015C

004E7BE8 8A9E5C010000 mov bl, byte ptr [esi+$015C]
004E7BEE 84DB test bl, bl
004E7BF0 7423 jz 004E7C15 <---- 判断

* Reference to pointer to GlobalVar_004FD478

004E7BF2 A150344F00 mov eax, dword ptr [$4F3450]

* Reference to field TProcessRegister.OFFS_0154

004E7BF7 8B9654010000 mov edx, [esi+$0154]

* Reference to: system.@LStrAsg;

004E7BFD E822C1F1FF call 00403D24

* Reference to pointer to GlobalVar_004FD478

004E7C02 A150344F00 mov eax, dword ptr [$4F3450]
004E7C07 83C004 add eax, +$04

* Reference to field TProcessRegister.OFFS_0158

004E7C0A 8B9658010000 mov edx, [esi+$0158]

* Reference to: system.@LStrAsg;

004E7C10 E80FC1F1FF call 00403D24
004E7C15 84DB test bl, bl
004E7C17 7512 jnz 004E7C2B <---- 这里可跳过 004E7C29 处的 JMP，再向上看。
004E7C19 8BC6 mov eax, esi

004E7C1B E8EC690000 call 004EE60C

* Reference to pointer to GlobalVar_004FD478

004E7C20 8B1550344F00 mov edx, [$4F3450]

* Reference to field GlobalVar_004FD478.OFFS_000C

004E7C26 88420C mov [edx+$0C], al
004E7C29 EB4D jmp 004E7C78 <---- 如果来到这里就玩完了！：（
004E7C2B 8D45F4 lea eax, [ebp-$0C]
004E7C2E 50 push eax

* Reference to pointer to GlobalVar_004FD478

004E7C2F A150344F00 mov eax, dword ptr [$4F3450]
004E7C34 8B00 mov eax, [eax]
004E7C36 8945DC mov [ebp-$24], eax
004E7C39 C645E00B mov byte ptr [ebp-$20], $0B
004E7C3D B8BC7C4E00 mov eax, $004E7CBC
```



```
004E7C42  8945E4             mov     [ebp-$1C], eax
004E7C45  C645E80B           mov     byte ptr [ebp-$18], $0B

* Reference to pointer to GlobalVar_004FD478

004E7C49  A150344F00         mov     eax, dword ptr [$4F3450]

* Reference to field GlobalVar_004FD478.OFFS_0004

004E7C4E  8B4004             mov     eax, [eax+$04]
004E7C51  8945EC             mov     [ebp-$14], eax
004E7C54  C645F00B           mov     byte ptr [ebp-$10], $0B
004E7C58  8D55DC             lea     edx, [ebp-$24]
004E7C5B  B902000000         mov     ecx, $00000002

* Possible String Reference to: '本软件受权给: %s%s 注册码: %s'           <---- 猫腻果然在这里！

向上看看怎么办才可以来到这里。

004E7C60  B8C87C4E00         mov     eax, $004E7CC8

......

--------------------------------------------------------------------------------

***** TRY

004EE53B  64FF30             push    dword ptr fs:[eax]
004EE53E  648920             mov     fs:[eax], esp

* Reference to field TProcessRegister.OFFS_0158

004EE541  8B8658010000       mov     eax, [esi+$0158]

* Reference to: system.@LStrLen:Integer;                               <---- LStrLen:Integer 注册码长度

004EE547  E8045AF1FF         call    00403F50
004EE54C  83F812             cmp     eax, +$12                         <---- 必须是 12h=18 位，此处证明注册码格式为
004EE54F  740C               jz      004EE55D                             xxx-xxxxx-xxxx-xxx

* Reference to field TProcessRegister.OFFS_015C

004EE551  C6865C01000000     mov     byte ptr [esi+$015C], $00
004EE558  E981000000         jmp     004EE5DE
004EE55D  8D45FC             lea     eax, [ebp-$04]
004EE560  50                 push    eax
004EE561  B903000000         mov     ecx, $00000003
004EE566  BA10000000         mov     edx, $00000010

* Reference to field TProcessRegister.OFFS_0158

004EE56B  8B8658010000       mov     eax, [esi+$0158]

* Reference to: system.@LStrCopy;

004EE571  E8E25BF1FF         call    00404158
004EE576  33D2               xor     edx, edx
004EE578  8B45FC             mov     eax, [ebp-$04]

* Reference to: sysutils.StrToIntDef(System.AnsiString;System.Integer):System.Integer;

004EE57B  E828AEF1FF         call    004093A8
004EE580  8945F8             mov     [ebp-$08], eax
004EE583  837DF800           cmp     dword ptr [ebp-$08], +$00
004EE587  7509               jnz     004EE592

* Reference to field TProcessRegister.OFFS_015C

004EE589  C6865C01000000     mov     byte ptr [esi+$015C], $00
004EE590  EB4C               jmp     004EE5DE
004EE592  33C0               xor     eax, eax
004EE594  8945F4             mov     [ebp-$0C], eax
004EE597  BF01000000         mov     edi, $00000001

* Reference to field TProcessRegister.OFFS_0158

004EE59C  8B8658010000       mov     eax, [esi+$0158]                 ----|
004EE5A2  8A5C38FF           mov     bl, byte ptr [eax+edi-$01]        |
004EE5A6  8BC3               mov     eax, ebx                          |
004EE5A8  04D0               add     al, -$30                          |
004EE5AA  2C0A               sub     al, $0A                            |
004EE5AC  731B               jnb     004EE5C9                          |
004EE5AE  8D45F0             lea     eax, [ebp-$10]                     |
                                                                        |
* Reference to field TProcessRegister.OFFS_0158                         |
                                                                        | 一句一句说太麻烦，总而言之一句话：
004EE5B1  8B9658010000       mov     edx, [esi+$0158]                 |
004EE5B7  8BD3               mov     edx, ebx                         |--- 第四部分注册码等于前三部分注册码的数字之和
                                                                        |
* Reference to: system.@LStrFromChar(String;Char);                       |    自己捉摸吧，很简单的。 *^_^*
                                                                        |
004EE5B9  E8BA58F1FF         call    00403E78                           |
004EE5BE  8B45F0             mov     eax, [ebp-$10]                     |
                                                                        |
* Reference to: sysutils.StrToInt(System.AnsiString):System.Integer;     |
                                                                        |
004EE5C1  E862ADF1FF         call    00409328                           |
004EE5C6  0145F4             add     [ebp-$0C], eax                     |
004EE5C9  47                 inc     edi                                 |
004EE5CA  83FF0F             cmp     edi, +$0F                           |
004EE5CD  75CD               jnz     004EE59C                       ----|
004EE5CF  8B45F4             mov     eax, [ebp-$0C]
004EE5D2  3B45F8             cmp     eax, [ebp-$08]
004EE5D5  7407               jz      004EE5DE

--------------------------------------------------------------------------------

不管怎样，写到这里就告一段落了。教程可真难写，尤其是排版。。。。。

..... 4dd466d0e1e0cc123afbe504da2462ca
```
