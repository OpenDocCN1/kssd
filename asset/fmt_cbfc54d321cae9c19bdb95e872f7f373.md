
# 汇编代码片段：硬盘序列号处理

```
004F0247  |>  8B45 FC         mov eax,[local.1]                     ;  硬盘物理序列号  
004F024A  |.  E8 414AF1FF     call NetUSB_e.00404C90               ;  System.@LStrLen(String):Integer;  
004F024F  |.  3BD8           cmp ebx,eax  
004F0251  |.  7F 04          jg short NetUSB_e.004F0257  
004F0253  |.  85DB          test ebx,ebx  
004F0255  |.^ 7F B9          \jg short NetUSB_e.004F0210           ;  i<length(HDSerailNo)继续循环  
004F0257  |>  8B5E 60        mov ebx,dword ptr ds:[esi+60]  
004F025A  |.  85DB          test ebx,ebx  
004F025C  |.  7F 11          jg short NetUSB_e.004F026F  
004F025E  |.  FF75 F4       push [local.3]                          ; /Arg2  
004F0261  |.  FF75 F0       push [local.4]                          ; |Arg1  
004F0264  |.  8BD7          mov edx,edi                             ; |  
004F0266  |.  33C0          xor eax,eax                             ; |  
004F0268  |.  E8 BF96F1FF   call NetUSB_e.0040992C                 ; \SysUtils.IntToHex(Int64;Integer):AnsiString;overload;  
004F026D  |.  EB 26         jmp short NetUSB_e.004F0295  
004F026F  |>  FF75 F4       push [local.3]                          ; /Arg2  
004F0272  |.  FF75 F0       push [local.4]                          ; |Arg1  
004F0275  |.  8BD7          mov edx,edi                             ; |  
004F0277  |.  8BC3          mov eax,ebx                              ; |  
004F0279  |.  E8 AE96F1FF   call NetUSB_e.0040992C                 ; \SysUtils.IntToHex(Int64;Integer):AnsiString;overload;  
004F027E  |.  8B07          mov eax,dword ptr ds:[edi]  
004F0280  |.  E8 0B4AF1FF   call NetUSB_e.00404C90                 ;  System.@LStrLen(String):Integer;  
004F0285  |.  8BC8          mov ecx,eax  
004F0287  |.  2B4E 60       sub ecx,dword ptr ds:[esi+60]  
004F028A  |.  8B56 60       mov edx,dword ptr ds:[esi+60]  
004F028D  |.  42             inc edx  
004F028E  |.  8BC7          mov eax,edi  
004F0290  |.  E8 934CF1FF   call NetUSB_e.00404F28                 ;  System.@LStrDelete;  
004F0295  |>  33C0          xor eax,eax  
004F0297  |.  5A             pop edx  
004F0298  |.  59             pop ecx  
004F0299  |.  59             pop ecx  
004F029A  |.  64:8910       mov dword ptr fs:[eax],edx  
004F029D  |.  68 B2024F00   push NetUSB_e.004F02B2  
004F02A2  |>  8D45 FC       lea eax,[local.1]  
004F02A5  |.  E8 1E47F1FF   call NetUSB_e.004049C8  
004F02AA  \.  C3             retn                                    ;  转换结束  
```

好像是硬盘物理序列号参与一个大数运算，用 `Delphi` 表示如下：

```delphi
Function SerialNumber(Str: String): String; //硬盘序列号转换函数
Var
  i: Integer;
  X: Int64;
Begin
  X := 0;
  For i := Length(Str) Downto 1 Do
  Begin
    X := X + $0A73A485F1C981EE;
    X := X - $459 Mod Ord(Str[i]);
  End;
  Result := IntToHex(X, 16);
  If Length(Str) = 0 Then Result := #0;
End;
```

`SerialNumber(硬盘物理序列号)=HDSerialNumber`，再在中间插入产品名称“网络\*\*”，仔细看看 `004EF382` ～`004EF3F2` 的代码，你发现了什么? 和 `call NetUSB_e.004F01C8` 函数一样, 有点郁闷，这是软件作者写了两次同样的代码，还是 `OD` 的问题? 不管了，到此为止，我们已经有了注册过程中所有重要的函数了，归纳一下：

1. `DecodeStr(注册码)=SN1`
2. `SerialNumber(硬盘物理序列号)=HDSerialNumber`
3. `SerialNumber(HDSerialNumber 前半部分+'网络\*\*'+HDSerialNumber 后半部分)=SN2`

`if SN1=SN2 Then 注册成功!`

通过 `3`，可求的 `SN2`，那么通过 `DecodeStr()` 的反函数 `EncodeStr(SN2)`，就可以求的软件的注册码了!!!

`delphi` 注册机框架表示如下：

```delphi
Procedure TFrmMain.FormCreate(Sender: TObject);
Begin
  edt1.Text := Trim(GetIdeSerialNumber());   //GetIdeSerialNumber()获取硬盘物理序列号函数
End;

Procedure TFrmMain.edt1Change(Sender: TObject);
Var
  MachineNo: String;
Begin
  MachineNo := Trim(SerialNumber(edt1.Text));    //SerialNumber(硬盘物理序列号)=HDSerialNumber
  edt2.Text := MachineNo
  MachineNo := LeftStr(MachineNo, Length(MachineNo) Shr 1) + '网络\*\*' + RightStr(MachineNo, Length(MachineNo) -
  Length(MachineNo) Shr 1);
                                                    //SerialNumber(HDSerialNumber 前半部分+'网络\*\*'+HDSerialNumber 后半部分)
  MachineNo := Trim(SerialNumber(MachineNo));
  edt3.Text := EncodeStr(MachineNo);    //EncodeStr(SN2) 注册码
End;
```

说明: 因为在我的笔记本上无法正确显示机器码, 不知道机器码是什么形式显示的, 所有我就直接安装软件作者的思想, 直接从硬盘物理序列号求注册码了, 要是能正确显示的应该是 `HDSerialNumber`，可能在中间加点 `'-'` 如 `AAA-BBB-CCCCCCC` 等样子。

如果你是个懒人话, 背上 `TNT`, 跟我来，暴了它……

## 第一处: 重启验证

```
005AEBAD  |.  B8 5CEE5A00     mov eax,NetUSB_e.005AEE5C              ;  ASCII "qvFntL"
005AEBB2  >|.  E8 E98FFEFF    call NetUSB_e.00597BA0               ;  ->:THttpCli._PROC_00597BA0()
005AEBB7  |.  8B45 F4         mov eax,[local.3]                     ;  '注册'
005AEBBA  |.  5A             pop edx
005AEBBB  >|.  E8 0C64E5FF    call NetUSB_e.00404FCC               ;  ->System.@LStrPos;
005AEBC0  |.  85C0           test eax,eax
005AEBC2  |.  0F8F 2F020000  jg NetUSB_e.005AEDF7
005AEBC8  |.  33D2           xor edx,edx
005AEBCA  >|.  8B86 D0030000  mov eax,dword ptr ds:[esi+3D0]       ;  *PCl:N.A.
005AEBD0  |.  8B08           mov ecx,dword ptr ds:[eax]
005AEBD2  |.  FF91 C0010000  call dword ptr ds:[ecx+1C0]
005AEBD8  |.  8BC6           mov eax,esi
005AEBDA  |.  E8 91F0FFFF    call NetUSB_e.005ADC70
005AEBDF  >|.  8B86 64030000  mov eax,dword ptr ds:[esi+364]       ;  *MeKao:N.A.
005AEBE5  >|.  E8 9A06F4FF    call NetUSB_e.004EF284               ;  ->ActiveX.PROPSETHDR_OSVER_KIND(DWORD):Word;<+>
005AEBEA  |.  84C0           test al,al                             ;  真正爆破点!!
005AEBEC  |.  0F84 B6000000  je NetUSB_e.005AECA8
005AEBF2  |.  8D55 F0        lea edx,[local.4]
005AEBF5  >|.  8B86 64030000  mov eax,dword ptr ds:[esi+364]       ;  *MeKao:N.A.
005AEBFB  |.  8B40 54        mov eax,dword ptr ds:[eax+54]
005AEBFE  >|.  E8 E1A8E5FF    call NetUSB_e.004094E4               ;  ->SysUtils.Trim(AnsiString):AnsiString;overload;
005AEC03  |.  837D F0 00     cmp [local.4],0
005AEC07  |.  0F84 9B000000  je NetUSB_e.005AECA8
005AEC0D  |.  33D2           xor edx,edx                            ;  下面是隐藏注册菜单
005AEC0F  >|.  8B86 4C030000  mov eax,dword ptr ds:[esi+34C]       ;  *Pl_Web:N.A.
005AEC15  >|.  E8 72FFE9FF    call NetUSB_e.0044EB8C               ;  ->Controls.TControl.SetVisible(TControl;Boolean);
005AEC1A  |.  33D2           xor edx,edx
005AEC1C  >|.  8B86 5C030000  mov eax,dword ptr ds:[esi+35C]       ;  *N6:N.A.
005AEC22  >|.  E8 5D1FEBFF    call NetUSB_e.00460B84               ;  ->Menus.TMenuItem.SetVisible(TMenuItem;Boolean);
005AEC27  |.  33D2           xor edx,edx
005AEC29  >|.  8B86 CC030000  mov eax,dword ptr ds:[esi+3CC]       ;  *N36:N.A.
005AEC2F  >|.  E8 501FEBFF    call NetUSB_e.00460B84               ;  ->Menus.TMenuItem.SetVisible(TMenuItem;Boolean);
005AEC34  |.  8D55 EC        lea edx,[local.5]
005AEC37  |.  8BC6           mov eax,esi
005AEC39  >|.  E8 2E00EAFF    call NetUSB_e.0044EC6C               ;  ->Controls.TControl.GetText(TControl):TCaption;
005AEC3E  |.  8D45 EC        lea eax,[local.5]
```

## 第二处: 注册验证

```
005A4118  |.  A1 6C285C00     mov eax,dword ptr ds:[5C286C]
005A411D  |.  8B00           mov eax,dword ptr ds:[eax]
005A411F  |.  8B80 64030000  mov eax,dword ptr ds:[eax+364]
005A4125  |.  59             pop ecx                                   ;  '用户名'
005A4126  |.  E8 91B5F4FF    call NetUSB_e.004EF6BC                 ;  注册验证,关键!
005A412B  |.  84C0           test al,al
005A412D  |.  75 09          jnz short NetUSB_e.005A4138            ;  验证通不过,调用 Halt 结束程序。
005A412F  |.  33C0           xor eax,eax
005A4131  >|.  E8 D607E6FF    call NetUSB_e.0040490C                 ;  ->System.@Halt(Integer);
005A4136  |.  EB 59          jmp short NetUSB_e.005A4191
```

………………………………………………………………………………………………………………………………………………
