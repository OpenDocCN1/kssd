
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
