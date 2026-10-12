
```
0058962D  E8CAA7E7FF                call    00403DFC

00589632  C3                          ret

00589633  E998A1E7FF                jmp     004037D0

00589638  EBE8                      jmp     00589622

****** END
```

|

```
0058963A  5F                          pop     edi

0058963B  5E                          pop     esi

0058963C  5B                          pop     ebx

0058963D  8BE5                      mov     esp, ebp

0058963F  5D                          pop     ebp

00589640  C3                          ret
```

> **patch:**
>
> 1、去除 14 天使用限制：
>
> 将`0058949B`处的指令：`E8006BFAFF call    0052FFA0` 改为：`xor eax,eax nop nop nop`，表示已经使用天数为 0
>
> 2、使程序启动时 About 对话框由 Nag screen 变成 splash screen。
>
> 将指令`005894A8  C6405D00                  mov     byte ptr [eax+$5D], $00`
>
> 改成`01`

> 来看看 Dede 生成的`dpr`文件：

```
begin

{
006A4D90  55                          push    ebp
006A4D91  8BEC                      mov     ebp, esp
006A4D93  83C4F4                    add     esp, -$0C
006A4D96  53                          push    ebx
006A4D97  B870456A00                mov     eax, $006A4570

* Reference to: sysinit.@InitExe;
|
006A4D9C  E84720D6FF                call    00406DE8

* Reference to TApplication instance
|
006A4DA1  8B1DA8F76A00              mov     ebx, [$6AF7A8]
006A4DA7  8B0B                      mov     ecx, [ebx]
006A4DA9  B201                      mov     dl, $01

* Reference to class TDM
|
006A4DAB  A1BC905800                mov     eax, dword ptr [$5890BC]

* Reference to: forms.TCustomForm.Create(TCustomForm;boolean;Classes.TComponent);
|       or: forms.TDataModule.Create(TDataModule;boolean;Classes.TComponent);
|
006A4DB0  E83BEADAFF                call    004537F0

* Reference to TDM instance
|
006A4DB5  8B15FCF76A00              mov     edx, [$6AF7FC]
006A4DBB  8902                      mov     [edx], eax
006A4DBD  8B0B                      mov     ecx, [ebx]
006A4DBF  B201                      mov     dl, $01

* Reference to class TFrmAbout
|
006A4DC1  A1A82F5900                mov     eax, dword ptr [$592FA8]

* Reference to: forms.TCustomForm.Create(TCustomForm;boolean;Classes.TComponent);
|       or: forms.TDataModule.Create(TDataModule;boolean;Classes.TComponent);
|
006A4DC6  E8719EDAFF                call    0044EC3C

* Reference to TFrmAbout instance
|
006A4DCB  8B1578F46A00              mov     edx, [$6AF478]
006A4DD1  8902                      mov     [edx], eax

* Reference to TDM instance
|
006A4DD3  A1FCF76A00                mov     eax, dword ptr [$6AF7FC]
006A4DD8  8B00                      mov     eax, [eax]

* Reference to field TDM.OFFS_005D
|       是注册版本？
006A4DDA  80785D00                  cmp     byte ptr [eax+$5D], $00 ;0 为 demo 版
006A4DDE  7425                      jz      006A4E05

如果是正式版,则检查有无命令行参数，如果有，
不显示 About 对话框，如果无命令行参数，则
显示 about 对话框，但是于 demo 版不同，about 对话
框的 ok 按钮不可见，about 对话框作 splash screen
用，进入程序主界面后 about 对话框自动消失。

* Reference to: system.ParamCount:Integer;
|
006A4DE0  E85FDBD5FF                call    00402944
006A4DE5  48                          dec     eax
006A4DE6  7F52                      jnle    006A4E3A

* Reference to TFrmAbout instance
|
006A4DE8  A178F46A00                mov     eax, dword ptr [$6AF478]
006A4DED  8B00                      mov     eax, [eax]

* Reference to: forms.TCustomForm.Show(TCustomForm);
|
006A4DEF  E8E0DDDAFF                call    00452BD4

* Reference to TFrmAbout instance
|
006A4DF4  A178F46A00                mov     eax, dword ptr [$6AF478]
006A4DF9  8B00                      mov     eax, [eax]
006A4DFB  8B10                      mov     edx, [eax]

* Possible reference to virtual method TFrmAbout.OFFS_0080
|
006A4DFD  FF9280000000              call    dword ptr [edx+$0080]
006A4E03  EB35                      jmp     006A4E3A

* Reference to TFrmAbout instance
|
demo 版转此，使 about 对话框上的 ok 按钮可见，用户必须
点击 OK 按钮后，才能进入程序主界面。这里 ABout 对话框
就相当于 nag screen.
|
006A4E05  A178F46A00                mov     eax, dword ptr [$6AF478]
006A4E0A  8B00                      mov     eax, [eax]

* Reference to control TFrmAbout.BtnOK : TButton
|
006A4E0C  8B80D4020000              mov     eax, [eax+$02D4]
006A4E12  B201                      mov     dl, $01

* Reference to: controls.TControl.SetVisible(TControl;System.Boolean);
|
006A4E14  E8D326D9FF                call    004374EC

* Reference to TFrmAbout instance
|
006A4E19  A178F46A00                mov     eax, dword ptr [$6AF478]
006A4E1E  8B00                      mov     eax, [eax]
006A4E20  8B10                      mov     edx, [eax]

* Possible reference to virtual method TFrmAbout.OFFS_00D8
|
006A4E22  FF92D8000000              call    dword ptr [edx+$00D8]

* Reference to TDM instance
|
006A4E28  A1FCF76A00                mov     eax, dword ptr [$6AF7FC]
006A4E2D  8B00                      mov     eax, [eax]

* Reference to field TDM.OFFS_006C
|
006A4E2F  83786C00                  cmp     dword ptr [eax+$6C], +$00
006A4E33  7D05                      jnl     006A4E3A

* Reference to: system.@Halt0;
|
006A4E35  E8C6EDD5FF                call    00403C00

程序开始正常初始化

006A4E3A  8B03                      mov     eax, [ebx]

* Reference to: forms.TApplication.Initialize(TApplication);
|       or: webbroker.TWebApplication.Initialize(TWebApplication);
|
006A4E3C  E87F15DBFF                call    004563C0
006A4E41  8B03                      mov     eax, [ebx]

==========================================================================
```

> 再来看看程序是如何得到软件使用天数的吧：

```
:0052FFA0  55                            push ebp
:0052FFA1  8BEC                          mov ebp, esp

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0052FF7C(C)
|
:0052FFA3 B905000000                      mov ecx, 00000005

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0052FFAD(C)
|
:0052FFA8 6A00                          push 00000000
:0052FFAA 6A00                          push 00000000
:0052FFAC 49                              dec ecx
:0052FFAD 75F9                          jne 0052FFA8
:0052FFAF 51                              push ecx
:0052FFB0 53                              push ebx
:0052FFB1 56                              push esi
:0052FFB2 57                              push edi
:0052FFB3 8955FC                          mov dword ptr [ebp-04], edx
:0052FFB6 8945F8                          mov dword ptr [ebp-08], eax
:0052FFB9 33C0                          xor eax, eax
:0052FFBB 55                              push ebp

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0052FF8D(C)
|
:0052FFBC 6846015300                      push 00530146
:0052FFC1 64FF30                          push dword ptr fs:[eax]
:0052FFC4 648920                          mov dword ptr fs:[eax], esp

:0052FFC7 C745F446000000                  mov [ebp-0C], 00000046
创建 TRegistry 对象
:0052FFCE B201                          mov dl, 01
:0052FFD0 A1388C4600                      mov eax, dword ptr [00468C38]    ;TRegistry
:0052FFD5 E8CA8DF3FF                      call 00468DA4
```
