
# 快乐传说脱机 破解分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-1.htm`](https://www.pediy.com/kssd/pediy09/pediy09-1.htm)

> 【文章标题】: 快乐传说脱机 破解分析  
> 【文章作者】: goodcode  
> 【软件名称】: 快乐传说脱机 1.39  
> 【下载地址】: 自己搜索下载  
> 【加壳方式】: `ASPack 2.12 -> Alexey Solodovnikov`  
> 【保护方式】: 网络验证  
> 【编写语言】: `Delphi`  
> 【使用工具】: `OD` + `DEDE` + `IDA`  
> 【操作平台】: `Win32`  
> 【软件介绍】: 一款游戏的脱机外挂  
> 【作者声明】: 外挂已经停止更新,并且不能正常连接验证服务器.  
>
> ----------------------------------------------------------------------  
> 【详细过程】  

打开程序通过 `"快乐传说脱机.exe"` 打开 `"gjmain.dat"`, 确定外挂主程序为 `"gjmain.dat"`.  

将 `"gjmain.dat"` 改名为 `"gjmain.exe"`, `peid` 查壳为 `"ASPack 2.12 -> Alexey Solodovnikov"`, 通过脱壳机自动脱壳.  

启动 `"gjmain.exe"`, 输入帐号密码日志窗口提示如下，请稍等)......  
7:27:55 连接验证服务器失败  
7:28:01 登陆成功  
可以进入游戏,但是挂机功能不能正常使用(不能寻怪)  

通过 `dede` 分析 `"gjmain.exe"`, 然后创建项目文件.  

打开 `"main.dfm"`, 游览窗口上的组件发现 `ClientSocket1` 很有可能用于验证, 内容如下  

```
  object ClientSocket1: TClientSocket
    Active = False
    ClientType = ctNonBlocking
    Host = 'www.csjpwg.com'
    Port = 80
    OnConnect = ClientSocket1Connect
    OnRead = ClientSocket1Read
    OnError = ClientSocket1Error
    Left = 177
    Top = 417
  end
```

打开 `"main.pas"`, 查找上面组件相关的事件代码  

```
  procedure TGjForm.ClientSocket1Connect(Sender : TObject);
  begin
  (*
  004EDB28   53                       push    ebx
  004EDB29   8BD8                   mov     ebx, eax
  004EDB2B   A154234F00             mov     eax, dword ptr [$004F2354]
  004EDB30   833800                 cmp     dword ptr [eax], +$00
  004EDB33   7423                   jz      004EDB58
  004EDB35   8B1554234F00           mov     edx, [$004F2354]
  004EDB3B   8B12                   mov     edx, [edx]

  * Reference to control ClientSocket1 : N.A.
  |
  004EDB3D   8B8378040000           mov     eax, [ebx+$0478]
  004EDB43   8B8090000000           mov     eax, [eax+$0090]

  * Reference to: ScktComp.TCustomWinSocket.SendText(TCustomWinSocket;AnsiString):Integer;
  |
  004EDB49   E8BA8EFAFF             call    00496A08
  004EDB4E   A154234F00             mov     eax, dword ptr [$004F2354]

  * Reference to: System.@LStrClr(void;void);
  |
  004EDB53   E8B066F1FF             call    00404208
  004EDB58   5B                     pop     ebx
  004EDB59   C3                     ret

  *)
  end;

  procedure TGjForm.ClientSocket1Error(Sender : TObject);
  begin
  (*
  004EDB5C   55                     push    ebp
  004EDB5D   8BEC                   mov     ebp, esp

  * Possible String Reference to: '连接验证服务器失败'
  |
  004EDB5F   B980DB4E00             mov     ecx, $004EDB80
  004EDB64   BA00800000             mov     edx, $00008000
  004EDB69   A160464F00             mov     eax, dword ptr [$004F4660]

  |
  004EDB6E   E8C1D8FFFF             call    004EB434
  004EDB73   5D                     pop     ebp
  004EDB74   C20800                 ret     $0008

  *)
  end;
```

通过 `ClientSocket1Error` 确认这个组件与网络验证密切相关.  

查看 `ClientSocket1` 的属性 `Host` 与 `Port` 得知验证是用 web 方式进行的,现在 `"www.csjpwg.com"` 已经不能打开, 所以我们通过 `winhex` 修改 `"www.csjpwg.com"` 属性为 `"www.baidu.com"` 多余字节以 00 填充.  

输入帐号密码进入游戏,点击自动战斗发现依旧是不能寻怪.  
我们下面的重点放在 `ClientSocket1Read` 事件处理过程.  
因为已经不能连接原始的验证服务器,所以也不能跟踪出正确的验证路线,所以大部分也就只能猜测了-\_-...  
用 `ida` 展开 exe 文件, 来到 `ClientSocket1Read` 过程  

```
CODE:004EDB94 ClientSocket1Read proc near
CODE:004EDB94
CODE:004EDB94 var_20            = dword ptr -20h
CODE:004EDB94 struseredit2      = dword ptr -1Ch
CODE:004EDB94 var_18            = dword ptr -18h
CODE:004EDB94 var_14            = dword ptr -14h
CODE:004EDB94 strUseredit       = dword ptr -10h
CODE:004EDB94 var_C             = dword ptr -0Ch
CODE:004EDB94 RecvText          = dword ptr -8
CODE:004EDB94 CustomWinSocket   = dword ptr -4
CODE:004EDB94
CODE:004EDB94                   push    ebp
CODE:004EDB95                   mov     ebp, esp
CODE:004EDB97                   push    0
CODE:004EDB99                   push    0
CODE:004EDB9B                   push    0
CODE:004EDB9D                   push    0
CODE:004EDB9F                   push    0
CODE:004EDBA1                   push    0
CODE:004EDBA3                   push    0
CODE:004EDBA5                   push    0
CODE:004EDBA7                   push    ebx
CODE:004EDBA8                   push    esi
CODE:004EDBA9                   push    edi
CODE:004EDBAA                   mov     [ebp+CustomWinSocket], ecx
CODE:004EDBAD                   mov     ebx, eax        ; 保存 self 指针
CODE:004EDBAF                   xor     eax, eax
CODE:004EDBB1                   push    ebp
CODE:004EDBB2                   push    offset loc_4EDCD9
CODE:004EDBB7                   push    dword ptr fs:[eax]
CODE:004EDBBA                   mov     fs:[eax], esp
CODE:004EDBBD                   lea     edx, [ebp+RecvText]
CODE:004EDBC0                   mov     eax, [ebp+CustomWinSocket]
CODE:004EDBC3                   call    @Scktcomp@TCustomWinSocket@ReceiveText$qqrv ; 返回数据
CODE:004EDBC8                   mov     eax, [ebx+478h]
CODE:004EDBCE                   cmp     dword ptr [eax+34h], 50h
CODE:004EDBD2                   jz      short loc_4EDC14 ; 客户端执行时跳转
CODE:004EDBD4                   lea     eax, [ebp+var_C]
CODE:004EDBD7                   mov     edx, [ebp+RecvText]
CODE:004EDBDA                   call    @System@@LStrLAsg$qqrv ; System::__linkproc__ LStrLAsg(void)
CODE:004EDBDF                   lea     edx, [ebp+strUseredit]
CODE:004EDBE2                   mov     eax, [ebx+324h] ; useredit 控件
CODE:004EDBE8                   call    @TControl@GetText$qqrv ; TControl::GetText(void)
CODE:004EDBED                   mov     eax, [ebp+strUseredit]
CODE:004EDBF0                   push    eax
CODE:004EDBF1                   lea     edx, [ebp+var_14]
CODE:004EDBF4                   mov     eax, ebx
CODE:004EDBF6                   call    sub_4EB218
CODE:004EDBFB                   mov     eax, [ebp+var_14]
CODE:004EDBFE                   mov     ecx, [ebp+var_C]
CODE:004EDC01                   pop     edx
CODE:004EDC02                   call    sub_4E3C58
CODE:004EDC07                   mov     eax, [ebp+CustomWinSocket]
CODE:004EDC0A                   call    @Scktcomp@TCustomWinSocket@Close$qqrv ; Scktcomp::TCustomWinSocket::Close(void)
CODE:004EDC0F                   jmp     loc_4EDC99
CODE:004EDC14 ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CODE:004EDC14
CODE:004EDC14 loc_4EDC14:                     ; CODE XREF: ClientSocket1Read+3Ej
CODE:004EDC14                   mov     edx, [ebp+RecvText]
CODE:004EDC17                   mov     eax, offset dword_4EDCF0 ; 字符串 REG: 将此字符串前 3 字符串修改为 ver(因为 www.baidu.com 返回的数据中
CODE:004EDC17                                       ; 会包含这些字符串)
CODE:004EDC1C                   call    @System@@LStrPos$qqrv ; System::__linkproc__ LStrPos(void)
CODE:004EDC21                   mov     esi, eax        ; 在返回字符串中查找
CODE:004EDC23                   test    esi, esi
CODE:004EDC25                   jle     short loc_4EDC99 ; 如果发现字符串不跳转
CODE:004EDC27                   lea     eax, [ebp+var_C]
CODE:004EDC2A                   call    @System@@LStrClr$qqrr17System@AnsiString ; System::__linkproc__ LStrClr(System::AnsiString &)
CODE:004EDC2F                   add     esi, 4            ; 上面查找返回的 pos+4
```



```
CODE:004EDC32                   mov     eax, [ebp+RecvText]

CODE:004EDC35                   call    @System@_16823  ; System::_16823

CODE:004EDC3A                   mov     edi, eax        ; 取字符串内某成员

CODE:004EDC3C                   sub     edi, esi        ; 计算出一个长度

CODE:004EDC3E                   jl      short loc_4EDC69

CODE:004EDC40                   inc     edi             ; 循环 取数据然后连接吧

CODE:004EDC41

CODE:004EDC41 loc_4EDC41:                           ; CODE XREF: ClientSocket1Read+D3j

CODE:004EDC41                   mov     eax, [ebp+RecvText]

CODE:004EDC44                   cmp     byte ptr [eax+esi-1], 0Dh ; 判断当前 pos 是否指向 0dh 内容的字节

CODE:004EDC49                   jz      short loc_4EDC69 ; 指向 0dh 跳转

CODE:004EDC4B                   lea     eax, [ebp+var_18]

CODE:004EDC4E                   mov     edx, [ebp+RecvText]

CODE:004EDC51                   mov     dl, [edx+esi-1]

CODE:004EDC55                   call    unknown_libname_13 ; Borland Visual Component Library & Packages

CODE:004EDC5A                   mov     edx, [ebp+var_18]

CODE:004EDC5D                   lea     eax, [ebp+var_C] ; 接到这个字符串后

CODE:004EDC60                   call    @System@@LStrCat$qqrv ; System::__linkproc__ LStrCat(void)

CODE:004EDC65                   inc     esi             ; 接字符串

CODE:004EDC66                   dec     edi

CODE:004EDC67                   jnz     short loc_4EDC41

CODE:004EDC69

CODE:004EDC69 loc_4EDC69:                           ; CODE XREF: ClientSocket1Read+AAj

CODE:004EDC69                                             ; ClientSocket1Read+B5j

CODE:004EDC69                   lea     edx, [ebp+struseredit2]

CODE:004EDC6C                   mov     eax, [ebx+324h]

CODE:004EDC72                   call    @TControl@GetText$qqrv ; TControl::GetText(void)

CODE:004EDC77                   mov     eax, [ebp+struseredit2] ; 从某控件取字符串(Useredit)

CODE:004EDC7A                   push    eax

CODE:004EDC7B                   lea     edx, [ebp+var_20] ; 此变量还未初始化过

CODE:004EDC7E                   mov     eax, ebx        ; self 指针

CODE:004EDC80                   call    sub_4EB218      ; 功能未知 返回服务器登陆字符串的某一部分吧

CODE:004EDC85                   mov     eax, [ebp+var_20]

CODE:004EDC88                   mov     ecx, [ebp+var_C]

CODE:004EDC8B                   pop     edx

CODE:004EDC8C                   call    sub_4E3C58      ; 外挂验证检测函数

CODE:004EDC91                   mov     eax, [ebp+CustomWinSocket]

CODE:004EDC94                   call    @Scktcomp@TCustomWinSocket@Close$qqrv ; Scktcomp::TCustomWinSocket::Close(void)

CODE:004EDC99

CODE:004EDC99 loc_4EDC99:                           ; CODE XREF: ClientSocket1Read+7Bj

CODE:004EDC99                                             ; ClientSocket1Read+91j

CODE:004EDC99                   xor     eax, eax

CODE:004EDC9B                   pop     edx

CODE:004EDC9C                   pop     ecx

CODE:004EDC9D                   pop     ecx

CODE:004EDC9E                   mov     fs:[eax], edx

CODE:004EDCA1                   push    offset loc_4EDCE0

CODE:004EDCA6

CODE:004EDCA6 loc_4EDCA6:                           ; CODE XREF: ClientSocket1Read+14Aj

CODE:004EDCA6                   lea     eax, [ebp+var_20]

CODE:004EDCA9                   call    @System@@LStrClr$qqrr17System@AnsiString ; System::__linkproc__ LStrClr(System::AnsiString &)

CODE:004EDCAE                   lea     eax, [ebp+struseredit2]

CODE:004EDCB1                   call    @System@@LStrClr$qqrr17System@AnsiString ; System::__linkproc__ LStrClr(System::AnsiString &)

CODE:004EDCB6                   lea     eax, [ebp+var_18]

CODE:004EDCB9                   mov     edx, 2

CODE:004EDCBE                   call    @System@@LStrArrayClr$qqrv ; System::__linkproc__ LStrArrayClr(void)

CODE:004EDCC3                   lea     eax, [ebp+strUseredit]

CODE:004EDCC6                   call    @System@@LStrClr$qqrr17System@AnsiString ; System::__linkproc__ LStrClr(System::AnsiString &)

CODE:004EDCCB                   lea     eax, [ebp+var_C]

CODE:004EDCCE                   mov     edx, 2

CODE:004EDCD3                   call    @System@@LStrArrayClr$qqrv ; System::__linkproc__ LStrArrayClr(void)

CODE:004EDCD8                   retn

CODE:004EDCD9 ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CODE:004EDCD9

CODE:004EDCD9 loc_4EDCD9:                           ; DATA XREF: ClientSocket1Read+1Eo

CODE:004EDCD9                   jmp     @System@@HandleFinally$qqrv ; System::__linkproc__ HandleFinally(void)

CODE:004EDCDE ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CODE:004EDCDE                   jmp     short loc_4EDCA6

CODE:004EDCE0 ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CODE:004EDCE0

CODE:004EDCE0 loc_4EDCE0:                           ; DATA XREF: ClientSocket1Read+10Do

CODE:004EDCE0                   pop     edi

CODE:004EDCE1                   pop     esi

CODE:004EDCE2                   pop     ebx

CODE:004EDCE3                   mov     esp, ebp

CODE:004EDCE5                   pop     ebp

CODE:004EDCE6                   retn

CODE:004EDCE6 ClientSocket1Read endp

来到验证函数内

CODE:004E3C58 sub_4E3C58      proc near           ; CODE XREF: sub_4E26DC+4Ap

CODE:004E3C58                                             ; ClientSocket1Read+6Ep ...

CODE:004E3C58

CODE:004E3C58 var_18          = dword ptr -18h

CODE:004E3C58 var_14          = dword ptr -14h

CODE:004E3C58 var_10          = dword ptr -10h

CODE:004E3C58 var_C           = dword ptr -0Ch

CODE:004E3C58 var_8           = dword ptr -8

CODE:004E3C58 var_4           = dword ptr -4

CODE:004E3C58

CODE:004E3C58                   push    ebp

CODE:004E3C59                   mov     ebp, esp

CODE:004E3C5B                   add     esp, 0FFFFFFE8h

CODE:004E3C5E                   push    ebx

CODE:004E3C5F                   push    esi

CODE:004E3C60                   push    edi

CODE:004E3C61                   xor     ebx, ebx

CODE:004E3C63                   mov     [ebp+var_14], ebx

CODE:004E3C66                   mov     [ebp+var_18], ebx

CODE:004E3C69                   mov     [ebp+var_10], ebx

CODE:004E3C6C                   mov     [ebp+var_C], ecx

CODE:004E3C6F                   mov     [ebp+var_8], edx

CODE:004E3C72                   mov     [ebp+var_4], eax

CODE:004E3C75                   mov     eax, [ebp+var_4]

CODE:004E3C78                   call    @System@@LStrAddRef$qqrv ; System::__linkproc__ LStrAddRef(void)

CODE:004E3C7D                   mov     eax, [ebp+var_8]

CODE:004E3C80                   call    @System@@LStrAddRef$qqrv ; System::__linkproc__ LStrAddRef(void)

CODE:004E3C85                   mov     eax, [ebp+var_C]

CODE:004E3C88                   call    @System@@LStrAddRef$qqrv ; System::__linkproc__ LStrAddRef(void)

CODE:004E3C8D                   xor     eax, eax

CODE:004E3C8F                   push    ebp

CODE:004E3C90                   push    offset sub_4E3D73

CODE:004E3C95                   push    dword ptr fs:[eax]

CODE:004E3C98                   mov     fs:[eax], esp

CODE:004E3C9B                   push    [ebp+var_4]

CODE:004E3C9E                   push    [ebp+var_8]

CODE:004E3CA1                   lea     edx, [ebp+var_18]

CODE:004E3CA4                   mov     eax, ds:off_4F2404
```



```
 CODE:004E3CA9             mov     eax, [eax]     ; 这是堆栈上能看到登陆帐号,服务器名称等信息  
 CODE:004E3CAB             call    @TControl@GetText$qqrv ; TControl::GetText(void)  
 CODE:004E3CB0             push    [ebp+var_18]  
 CODE:004E3CB3             lea     eax, [ebp+var_14]  
 CODE:004E3CB6             mov     edx, 3  
 CODE:004E3CBB             call    sub_404588     ; LStrCat  
 CODE:004E3CC0             mov     eax, [ebp+var_14]  
 CODE:004E3CC3             call    sub_48F58C     ; TIdTCPClient._PROC_0048F58C()  
 CODE:004E3CC8             imul    ebx, eax, 3039h  
 CODE:004E3CCE             mov     edx, ds:dword_4F4604  
 CODE:004E3CD4             mov     eax, ebx  
 CODE:004E3CD6             call    sub_48F5F0     ; * Reference to : TIdTCPClient._PROC_0048F5F0()  
 CODE:004E3CDB             imul    eax, 3039h  
 CODE:004E3CE1             mov     ebx, eax  
 CODE:004E3CE3             lea     ecx, [ebp+var_10]  
 CODE:004E3CE6             mov     edx, [ebp+var_C]  
 CODE:004E3CE9             mov     eax, ebx        ; 执行到这里会异常 也许是上面的字符串有问题吧  
 CODE:004E3CEB             call    @Dbclient@TClientDataSet@CreateDSCursor$qqr44System@_DelphiInterface$t16Dsintf@IDSCursor_ ; * Reference to : TIdTCPClient._PROC_0048F694() ;nop 掉这里  
 CODE:004E3CF0             mov     ds:dword_4F4608, 0FFFFFFFFh ; 一处验证标志 >0 就算通过验证吧  
 CODE:004E3CFA             xor     eax, eax  
 CODE:004E3CFC             push    ebp  
 CODE:004E3CFD             push    offset loc_4E3D1F  
 CODE:004E3D02             push    dword ptr fs:[eax]  
 CODE:004E3D05             mov     fs:[eax], esp  
 CODE:004E3D08             mov     eax, [ebp+var_10] ; 这里装的应该是含有数字的字符串  
 CODE:004E3D0B             call    @Sysutils@StrToInt$qqrx17System@AnsiString ; 修改为 mov eax, 1 替换掉这个 call  
 CODE:004E3D10             mov     ds:dword_4F4608, eax ; 一处验证标志 >0 就算通过验证吧  
 CODE:004E3D15             xor     eax, eax  
 CODE:004E3D17             pop     edx  
 CODE:004E3D18             pop     ecx  
 CODE:004E3D19             pop     ecx  
 CODE:004E3D1A             mov     fs:[eax], edx  
 CODE:004E3D1D             jmp     short loc_4E3D30  
 CODE:004E3D1F ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  
 CODE:004E3D1F  
 CODE:004E3D1F loc_4E3D1F:                     ; DATA XREF: sub_4E3C58+A5o  
 CODE:004E3D1F             jmp     sub_4038D4  
 CODE:004E3D24 ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  
 CODE:004E3D24             call    @@DoneExcept$qqrv ; __linkproc__ DoneExcept(void)  
 CODE:004E3D29             jmp     short loc_4E3D50  
 CODE:004E3D2B ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  
 CODE:004E3D2B             call    @@DoneExcept$qqrv ; __linkproc__ DoneExcept(void)  
 CODE:004E3D30  
 CODE:004E3D30 loc_4E3D30:                     ; CODE XREF: sub_4E3C58+C5j  
 CODE:004E3D30             cmp     ds:dword_4F4608, 0 ; 一处验证标志 >0 就算通过验证吧  
 CODE:004E3D37             jl      short loc_4E3D50 ; >0  
 CODE:004E3D39             mov     eax, offset dword_4F4610  
 CODE:004E3D3E             mov     edx, [ebp+var_C]  
 CODE:004E3D41             call    @System@@LStrAsg$qqrv ; System::__linkproc__ LStrAsg(void)  
 CODE:004E3D46             mov     ds:dword_4F460C, 31589h  
 CODE:004E3D50  
 CODE:004E3D50 loc_4E3D50:                     ; CODE XREF: sub_4E3C58+D1j  
 CODE:004E3D50                              ; sub_4E3C58+DFj  
 CODE:004E3D50             xor     eax, eax  
 CODE:004E3D52             pop     edx  
 CODE:004E3D53             pop     ecx  
 CODE:004E3D54             pop     ecx  
 CODE:004E3D55             mov     fs:[eax], edx  
 CODE:004E3D58             push    offset loc_4E3D7A  
 CODE:004E3D5D  
 CODE:004E3D5D loc_4E3D5D:                     ; CODE XREF: CODE:004E3D78j  
 CODE:004E3D5D             lea     eax, [ebp+var_18]  
 CODE:004E3D60             call    @System@@LStrClr$qqrr17System@AnsiString ; System::__linkproc__ LStrClr(System::AnsiString &)  
 CODE:004E3D65             lea     eax, [ebp+var_14]  
 CODE:004E3D68             mov     edx, 5  
 CODE:004E3D6D             call    @System@@LStrArrayClr$qqrv ; System::__linkproc__ LStrArrayClr(void)  
 CODE:004E3D72             retn  
 CODE:004E3D72 sub_4E3C58      endp ; sp = -38h  
```

修改过以上 3 处后继续使用外挂登陆, 进入游戏点自动战斗.

已经可以正常战斗了,到此破解完成.

--------------------------------------------------------------------------------

### 经验总结

外挂是一次验证, 判断标志也非常少, 使用压缩壳加壳, 字符串明码出现这些对破解来说带来很多方便.

不知道会不会有烂人看完文章自己写个 `loader` 在加一个广告拿出去忽悠...

--------------------------------------------------------------------------------

### 版权声明

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007 年 01 月 02 日 7:49:59

`4dd466d0e1e0cc123afbe504da2462ca`
