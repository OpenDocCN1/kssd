
# AddFlow

> 原文：[`www.pediy.com/kssd/pediy03/forum800.htm`](https://www.pediy.com/kssd/pediy03/forum800.htm)

`http://www.lassalle.com` 提供了最新版的 `AddFlow`。

以前的版本未找到！如果那位高手知道以前的版本（因为网上的破解程序只支持到 028 版）能告诉我网址或是给我发一个破解版的（这个 `OCX` 也不大！）或是注册号，在下不胜感激！

含义明显的字符串：

```
// ------------- FUNCTION function103 -------------------------

    function function103()
        number lNumber0;
        number lNumber1;
        number lNumber2;
        string lString0;
        string lString1;
        string lString2;
    begin
001BCB:0013:        lString0 = "";
001BD3:0013:        lString1 = "Type your name below. You must also type the name of your company and the product serial number. If you are installing an evaluation version, this serial number must be \\"EVALUATION\\"";
label41: //Ref: 001DD4
001C94:0013:        string4 = string7;
001C9C:0013:        string5 = string8;
001CA4:0013:        string6 = string9;
001CAC:00B5:        function1(lString0, lString1, string4, string5, string6);
001CC3:0021:        lNumber0 = LAST_RESULT;
001CCB:0013:        string7 = string4;
001CD3:0013:        string8 = string5;
001CDB:00B5:        function108(string6);
001CE6:0021:        lNumber2 = LAST_RESULT;
001CEE:0128:        lNumber2 = lNumber2 = 0;
001D00:0022:        if (lNumber2 = 0) then
                        goto label42;
                    endif;
001D0E:0023:        StrCompare(string6, "EVALUATION");
001D20:0021:        lNumber2 = LAST_RESULT;
001D28:0128:        lNumber2 = lNumber2 != 0;
001D3A:0022:        if (lNumber2 = 0) then
                        goto label42;
                    endif;
001D48:0013:        lString2 = "INVALID SERIAL NUMBER! If it is an evaluation version, just enter \\"EVALUATION\\" in the serial number field.";
001DBA:0021:        lNumber1 = -65535;
001DC4:002A:        MessageBox(lString2, lNumber1);
001DCC:0013:        string9 = string6;
001DD4:002C:        goto label41;
label42: //Ref: 001D00  001D3A
001DDD:0013:        string9 = string6;
001DE5:012F:        return(lNumber0);
001DEC:00B8:        return;
    end;
//-----------------------------------------------------------------------
```

`function108` 是关键的，分析一下应该可以写出 keymaker：

```
// ------------- FUNCTION function108 --------------------------------------

    function function108(pString0)
        number lNumber0;
        number lNumber1;
        number lNumber2;
        number lNumber3;
        number lNumber4;
        number lNumber5;
        number lNumber6;
        number lNumber7;
        number lNumber8;
        string lString0;
        string lString1;
        string lString2;
        string lString3;
        string lString4;
        string lString5;
    begin
001E0C:002F:        StrLength(pString0);
001E11:0021:        lNumber5 = LAST_RESULT;
001E19:0021:        lNumber4 = 0;
001E23:0021:        lNumber3 = 0;
001E2D:011A:        lNumber7 = lNumber5 - 1;
label44: //Ref: 001EBB
001E3E:0128:        lNumber8 = lNumber3 <= lNumber7;
001E4E:0022:        if (lNumber8 = 0) then
                        goto label46;
                    endif;
001E5C:007A:        GetByte(lNumber8, pString0, lNumber3);
001E67:0128:        lNumber8 = lNumber8 != 32;
001E79:0022:        if (lNumber8 = 0) then
                        goto label45;
                    endif;
001E87:007A:        GetByte(lNumber8, pString0, lNumber3);
001E92:007B:        SetByte(lString0, lNumber4, lNumber8);
001E9D:0119:        lNumber4 = lNumber4 + 1;
label45: //Ref: 001E79
001EAE:0119:        lNumber3 = lNumber3 + 1;
001EBB:002C:        goto label44;
label46: //Ref: 001E4E
001EC4:0021:        lNumber2 = 0;
001ECE:0021:        lNumber6 = 3;
001ED8:0021:        lNumber3 = 0;
001EE2:011A:        lNumber7 = lNumber4 - 2;
label47: //Ref: 001F9A
001EF3:0128:        lNumber8 = lNumber3 <= lNumber7;
001F03:0022:        if (lNumber8 = 0) then
                        goto label50;
                    endif;
001F11:007A:        GetByte(lNumber8, lString0, lNumber3);
001F1C:011A:        lNumber8 = lNumber8 - 48;
001F29:011B:        lNumber8 = lNumber8 * lNumber6;
001F34:0119:        lNumber2 = lNumber2 + lNumber8;
001F3F:0123:        lNumber2 = lNumber2 % 10;
001F4C:0128:        lNumber8 = lNumber6 = 3;
001F5E:0022:        if (lNumber8 = 0) then
                        goto label48;
                    endif;
001F6C:0021:        lNumber6 = 2;
001F76:002C:        goto label49;
label48: //Ref: 001F5E
001F7F:0021:        lNumber6 = 3;
label49: //Ref: 001F76
001F8D:0119:        lNumber3 = lNumber3 + 1;
001F9A:002C:        goto label47;
label50: //Ref: 001F03
001FA3:011A:        lNumber7 = lNumber4 - 1;
001FB0:007A:        GetByte(lNumber7, lString0, lNumber7);
001FBB:011A:        lNumber1 = lNumber7 - 48;
001FC8:0128:        lNumber7 = lNumber2 != lNumber1;
001FD8:0022:        if (lNumber7 = 0) then
                        goto label51;
                    endif;
001FE6:0021:        lNumber0 = 0;
001FF0:002C:        goto label52;
label51: //Ref: 001FD8
001FF9:0030:        StrSub(lString1, lString0, 0, 4);
00200B:0030:        StrSub(lString2, lString0, 4, 4);
00201D:0030:        StrSub(lString3, lString0, 8, 4);
00202F:0030:        StrSub(lString4, lString0, 12, 4);
002041:0124:        lString5 = lString1 + " ";
00204D:0124:        lString5 = lString5 + lString2;
002058:0124:        lString5 = lString5 + " ";
002064:0124:        lString5 = lString5 + lString3;
00206F:0124:        lString5 = lString5 + " ";
00207B:0124:        string10 = lString5 + lString4;
002086:0021:        lNumber0 = 1;
label52: //Ref: 001FF0
002094:012F:        return(lNumber0);
00209B:00B8:        return;
    end;
//------------------------------------------------------------------------------
```

`lic` 的格式：

```
// ------------- FUNCTION function93 ------------------------------------

    function function93()
        number lNumber0;
        number lNumber1;
        number lNumber2;
        string lString0;
        string lString1;
        string lString2;
        string lString3;
    begin
001173:0023:        StrCompare(string6, "EVALUATION");
001185:0021:        lNumber2 = LAST_RESULT;
00118D:0128:        lNumber2 = lNumber2 = 0;
00119F:0022:        if (lNumber2 = 0) then
                        goto label23;
                    endif;
0011AD:012F:        return(0);
label23: //Ref: 00119F
0011BA:0013:        lString0 = "ADDFLOW3.LIC";
0011CE:0013:        lString1 = WINSYSDIR;
0011D6:0021:        lNumber1 = 3;
0011E0:0065:        OpenFileMode(lNumber1);
0011E5:0011:        CreateFile(lNumber0, lString1, lString0);
0011F0:0021:        lNumber2 = LAST_RESULT;
0011F8:0128:        lNumber2 = lNumber2 = 0;
00120A:0022:        if (lNumber2 = 0) then
                        goto label24;
                    endif;
001218:0012:        WriteLine(lNumber0, "AddFlow ActiveX Control");
001237:0012:        WriteLine(lNumber0, "Copyright (c) 1997-99 Lassalle Technologies");
00126A:0124:        lString3 = "License number: " + string10;
001285:0012:        WriteLine(lNumber0, lString3);
00128D:0012:        WriteLine(lNumber0, "This product is licensed to:");
0012B1:0124:        lString3 = "User: " + string4;
0012C2:0012:        WriteLine(lNumber0, lString3);
0012CA:0124:        lString3 = "Company: " + string5;
0012DE:0012:        WriteLine(lNumber0, lString3);
```



```
0012E6:0012:         WriteLine(lNumber0, " ");   

0012EF:001F:         CloseFile(lNumber0);   

label24: //Ref: 00120A   

0012F8:012F:         return(0);   

001301:00B8:         return;   

    end;
```

把下面的 `JZ` 改为 `JMP` 就不会有 `AboutBox` 了。

```
001B:10001FD8  8BCE                MOV       ECX,ESI   

001B:10001FDA  E8DF6A0100          CALL      10018ABE   

001B:10001FDF  85C0                TEST      EAX,EAX   

001B:10001FE1  7405                JZ        10001FE8   

001B:10001FE3  6A01                PUSH      01   

001B:10001FE5  58                  POP       EAX   

001B:10001FE6  EB02                JMP       10001FEA   

001B:10001FE8  33C0                XOR       EAX,EAX   

001B:10001FEA  5F                  POP       EDI   

001B:10001FEB  3BC3                CMP       EAX,EBX   

001B:10001FED  89865C040000        MOV       [ESI+0000045C],EAX   

001B:10001FF3  5B                  POP       EBX   

001B:10001FF4  7407                JZ        10001FFD                //这里   

001B:10001FF6  8BCE                MOV       ECX,ESI   

001B:10001FF8  E8A51B0000          CALL      10003BA2    //调用 AboutBox( )显示对话框   

001B:10001FFD  33C0                XOR       EAX,EAX   

001B:10001FFF  5E                  POP       ESI   

001B:10002000  C20400              RET       0004   

//---------------------------------------------------------------
```

相关的 COM 接口的定义部分：

```
.rdata:1001B8D8                    dd offset aAboutbox    ; "AboutBox"            //接口函数名   

.rdata:1001B8DC                    db 0D8h ; ?   

.rdata:1001B8DD                    db 0FDh ; ?   

.rdata:1001B8DE                    db 0FFh ;   

.rdata:1001B8DF                    db 0FFh ;   

.rdata:1001B8E0                    db     0 ;   

.rdata:1001B8E1                    db     0 ;   

.rdata:1001B8E2                    db     0 ;   

.rdata:1001B8E3                    db     0 ;   

.rdata:1001B8E4                    db     0 ;   

.rdata:1001B8E5                    db     0 ;   

.rdata:1001B8E6                    db     0 ;   

.rdata:1001B8E7                    db     0 ;   

.rdata:1001B8E8                    dd offset sub_10003BA2        //接口函数的入口地址   

//----------------------------------------------------------------------
```

接口函数 `AboutBox( )` 的实现部分，被多处调用：

```
.text:10003BA2 sub_10003BA2    proc near                   ; CODE XREF: sub_10001F6F+89p   

.text:10003BA2                                        ; sub_10003B7D+Ep   

.text:10003BA2                                        ; DATA XREF: ...   

.text:10003BA2                    mov     eax, offset loc_1001957C   

.text:10003BA7                    call    _EH_prolog   

.text:10003BAC                    sub     esp, 70h   

.text:10003BAF                    push    64h   

.text:10003BB1                    lea     ecx, [ebp-7Ch]   

.text:10003BB4                    call    sub_10003C07   

.text:10003BB9                    mov     eax, dword_10020B44   

.text:10003BBE                    and     dword ptr [ebp-4], 0   

.text:10003BC2                    mov     [ebp-1Ch], eax   

.text:10003BC5                    mov     eax, dword_10020B48   

.text:10003BCA                    push    offset unk_10020B4C   

.text:10003BCF                    lea     ecx, [ebp-14h]   

.text:10003BD2                    mov     [ebp-18h], eax   

.text:10003BD5                    call    ??4CString@@QAEABV0@ABV0@@Z ; CString::operator=(CString const &)   

.text:10003BDA                    push    offset unk_10020B50   

.text:10003BDF                    lea     ecx, [ebp-10h]   

.text:10003BE2                    call    ??4CString@@QAEABV0@ABV0@@Z ; CString::operator=(CString const &)   

.text:10003BE7                    lea     ecx, [ebp-7Ch]   

.text:10003BEA                    call    ?DoModal@CDialog@@UAEHXZ ; CDialog::DoModal(void)   

.text:10003BEF                    or      dword ptr [ebp-4], 0FFFFFFFFh   

.text:10003BF3                    lea     ecx, [ebp-7Ch]   

.text:10003BF6                    call    sub_10003C6D   

.text:10003BFB                    mov     ecx, [ebp-0Ch]   

.text:10003BFE                    mov     large fs:0, ecx   

.text:10003C05                    leave   

.text:10003C06                    retn   

.text:10003C06 sub_10003BA2    endp ; sp =  4  4dd466d0e1e0cc123afbe504da2462ca
```
