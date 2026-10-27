
```
Reference to: sysutils.ChangeFileExt(System.AnsiString;System.AnsiString):System.AnsiString;
 |
 00589516  E81504E8FF
call    00409930
 0058951B  8B8578FEFFFF
    mov    eax, [ebp+$FFFFFE78] ;->扩展名换后的路径和文件名
        查找这个文件
         我看了看，在我的机器上没这个文件
 \* Reference to: system.@LStrToPChar;
 |
 00589521  E812ADE7FF
          call    00404238
 00589526
50                    push
  eax
 \* Reference to: kernel32.FindFirstFileA()
 |
 00589527
E84CDAE7FF            call    00406F78
 0058952C  8BD8
  mov    ebx, eax
 0058952E  83FBFF
          cmp    ebx, -$01
 00589531
743B                  jz
  0058956E
 00589533  8D45E8
    lea    eax, [ebp-$18]
 00589536  50
                push    eax
 00589537  8D85ACFEFFFF          lea
eax, [ebp+$FFFFFEAC]
 0058953D  50
          push    eax
 \* Reference
to: kernel32.FileTimeToLocalFileTime()
 |
 0058953E  E825DAE7FF
          call    00406F68
 00589543
85C0                  test
  eax, eax
 00589545  7421
      jz      00589568
 00589547  8D45FC
              lea    eax, [ebp-$04]
 0058954A  50
    push    eax
 0058954B  8D45FE
          lea    eax, [ebp-$02]
 0058954E
50                    push
  eax
 0058954F  8D45E8
    lea    eax, [ebp-$18]
 00589552  50
                push    eax
 \* Reference to: kernel32.FileTimeToDosDateTime()
 |
 00589553
E808DAE7FF            call    00406F60
 00589558  85C0
  test    eax, eax
 0058955A  740C
            jz      00589568
0058955C  8B45FC                mov
  eax, [ebp-$04]
 \* Reference to: sysutils.FileDateToDateTime(System.Integer):System.TDateTime;
 |
 0058955F  E84009E8FF
call    00409EA4
 00589564  DD5DF0
        fstp    qword ptr [ebp-$10]
 00589567
9B                    wait
00589568  53
  push    ebx
 \* Reference to: kernel32.FindClose()
|
 00589569  E802DAE7FF            call
  00406F70
 0058956E  33C0
      xor    eax, eax
 00589570  5A
                pop    edx
00589571  59
  pop    ecx
 00589572  59
            pop    ecx
 00589573
648910                mov
fs:[eax], edx
 00589576  EB0A
      jmp    00589582
 \*\*\*\*\*\* EXCEPT
 |
 00589578  E99F9FE7FF            jmp
  0040351C
 \* Reference to: system.@DoneExcept;
 |
 0058957D
E8F6A2E7FF            call    00403878
 \*\*\*\*\*\* END
 |
 \* Reference to DM
 |
 00589582
8B45F8                mov
eax, [ebp-$08]
 \* Reference to field TDM.OFFS\_005C
 |
 00589585
C6405C01              mov    byte
ptr [eax+$5C], $01
 00589589  A17C816B00
  mov    eax, dword ptr [$6B817C]
 0058958E  83C00E
              add    eax, +$0E
  ;14天试用期-已经使用的天数(因为用负数表示)
                  ;得到剩余天数
 \* Reference to DM
 |
 00589591  8B55F8
        mov    edx, [ebp-$08]
 \* Reference
to field TDM.OFFS\_006C
 |
 00589594  89426C
        mov    [edx+$6C], eax    ;保存剩余天数
         又随机反跟踪，无聊：)
 00589597
B201                  mov
dl, $01
 \* Reference to DM
 |
 00589599  8B45F8
            mov    eax, [ebp-$08]
 \* Reference to : TDM.HasWi95()
 |
 0058959C  E8D7000000
          call    00589678
 005895A1
84C0                  test
  al, al
 005895A3  751F
      jnz    005895C4
 005895A5  B201
                mov    dl, $01
 \* Reference to DM
 |
 005895A7  8B45F8
          mov    eax, [ebp-$08]
 \*
Reference to : TDM.HasWiNT()
 |
 005895AA  E841010000
        call    005896F0
 005895AF  84C0
                test    al,
al
 005895B1  7511
  jnz    005895C4
 005895B3  33D2
            xor    edx, edx
 005895B5
33C0                  xor
eax, eax
 |
 005895B7  E8E469FAFF
    call    0052FFA0
 005895BC  84C0
              test    al, al
005895BE  7504
jnz    005895C4
 005895C0  33C0
          xor    eax, eax
 005895C2
EB02                  jmp
005895C6
 005895C4  B001
    mov    al, $01
 \* Reference to DM
 |
005895C6  8B55F8                mov
  edx, [ebp-$08]
 \* Reference to field TDM.OFFS\_005C
 |
005895C9  88425C                mov
  [edx+$5C], al    ;有softice时，此标志置1
 \* Reference
to DM
 |
 005895CC  8B45F8
    mov    eax, [ebp-$08]
 \* Reference to field TDM.OFFS\_006C
 |
 005895CF  8B406C
  mov    eax, [eax+$6C] ;eax=剩余天数
      剩余天数应该>=0 <=14
 005895D2  83F80E
              cmp    eax, +$0E
005895D5  7F04
jnle    005895DB
 005895D7  85C0
          test    eax, eax
 005895D9
7D0A                  jnl
005895E5
 \* Reference to DM
 |
 005895DB  8B45F8
            mov    eax, [ebp-$08]
 \* Reference to field TDM.OFFS\_006C
 |
 005895DE  C7406CFFFFFFFF
      mov    dword ptr [eax+$6C], $FFFFFFFF
        当前日期不能早于那个.dpl文件的日期
 \* Reference
to: sysutils.Date:System.TDateTime;
 |
 005895E5  E80618E8FF
          call    0040ADF0
 \* Reference
to: system.@INT;
 |
 005895EA  E88194E7FF
    call    00402A70
 005895EF  DBBD68FEFFFF
        fstp    tbyte ptr [ebp+$FFFFFE68]
 005895F5
9B                    wait
005895F6  DD45F0                fld
  qword ptr [ebp-$10]    ;.dpl文件的日期
 \* Reference
to: system.@INT;
 |
 005895F9  E87294E7FF
    call    00402A70
 005895FE  DBAD68FEFFFF
        fld    tbyte ptr [ebp+$FFFFFE68]
 00589604
DED9                  fcompp
00589606  DFE0
fstsw  ax
 00589608  9E
        sahf
 00589609  730A
            jnb    00589615
\* Reference to DM
 |
 0058960B  8B45F8
        mov    eax, [ebp-$08]
 \* Reference
to field TDM.OFFS\_006C
 |
 0058960E  C7406CFFFFFFFF
    mov    dword ptr [eax+$6C], $FFFFFFFF
 00589615
33C0                  xor
eax, eax
 00589617  5A
      pop    edx
 00589618  59
                pop    ecx
00589619  59
  pop    ecx
 0058961A  648910
        mov    fs:[eax], edx
 \*\*\*\*\*\* FINALLY
 |
 \* Possible String Reference to: '\_^[]?
 |
 0058961D
683A965800            push    $0058963A
 00589622  8D8574FEFFFF          lea
eax, [ebp+$FFFFFE74]
 00589628  BA0B000000
    mov    edx, $0000000B
 \* Reference to: system.@LStrArrayClr;
 |
 0058962D  E8CAA7E7FF
call    00403DFC
 00589632  C3
          ret
 00589633  E998A1E7FF
          jmp    004037D0
 00589638
EBE8                  jmp
00589622
 \*\*\*\*\*\* END
 |
 0058963A  5F
              pop    edi
 0058963B
5E                    pop
  esi
 0058963C  5B
      pop    ebx
 0058963D  8BE5
              mov    esp, ebp
0058963F  5D
  pop    ebp
 00589640  C3
                ret
```

patch:

1、去除14天使用限制：

将`0058949B`处的指令：`E8006BFAFF call 0052FFA0` 改为：`xor eax,eax nop nop`，其中`nop`表示已经使用天数为0。

2、使程序启动时About对话框由Nag screen变成splash screen。

将指令`005894A8`：
```
C6405D00              mov    byte
ptr [eax+$5D], $00
```
   ^^改成01

来看看Dede生成的dpr文件：

```
begin
{
006A4D90  55
                   push
 ebp
006A4D91  8BEC
   mov    ebp, esp
006A4D93  83C4F4
           add    esp, -$0C
006A4D96
53                    push
   ebx
006A4D97  B870456A00
 mov    eax, $006A4570
 \* Reference to: sysinit.@InitExe;
 |
006A4D9C  E84720D6FF            call
   00406DE8
 \* Reference to TApplication instance
 |
006A4DA1
8B1DA8F76A00          mov    ebx, [$6AF7A8]
006A4DA7  8B0B
   mov    ecx, [ebx]
006A4DA9  B201
             mov    dl, $01
 \*
Reference to class TDM
 |
006A4DAB  A1BC905800
       mov    eax, dword ptr [$5890BC]
 \* Reference
to: forms.TCustomForm.Create(TCustomForm;boolean;Classes.TComponent);
 |
        or: forms.TDataModule.Create(TDataModule;boolean;Classes.TComponent);
 |
006A4DB0  E83BEADAFF
 call    004537F0
 \* Reference to TDM instance
 |
006A4DB5
8B15FCF76A00          mov    edx, [$6AF7FC]
006A4DBB  8902
   mov    [edx], eax
006A4DBD  8B0B
             mov    ecx, [ebx]
006A4DBF
B201                  mov
 dl, $01
 \* Reference to class TFrmAbout
 |
006A4DC1  A1A82F5900
           mov    eax, dword ptr [$592FA8]
```
