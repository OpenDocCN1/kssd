
因为仅有一处出现`.chm`，所以可以断定这就是打开`idahelp.chm`帮助文件的函数。再双击交叉引用`sub_403EE0`，可得到如下代码：

```
.text:00404026 loc_404026:                      ; CODE XREF: sub_403EE0+13Dj

.text:00404026                mov     eax, offset byte_57F4C0
.text:0040402B                mov     edx, esi
.text:0040402D                call    sub_466020
```

由此可见`sub_466020`需要两个参数`eax`和`edx`（典型的 Delphi 调用）。`eax`的参数为`offset byte_57F4C0`，但`edx`的参数为`esi`不能确定其值是多少。我们可以在`.text:00404026 loc_404026:`处下断点（快捷键为`F2`）对其进行跟踪。单击菜单`Debugger->Start process`（或快捷键`F9`）对`idag.exe`进行调试，这时又会出现一个`Ida`图形界面（点击`Yes`关闭`Debugger warning`对话框，再点击`OK`关闭`About`对话框）。然后按`F1`，这时程序就停在断点`loc_404026:`处，观察寄存器`esi`发现其值为`0`。这样两个参数就定下来了。

如果想确认`sub_466020`的功能，可以单步跟踪到`sub_466020`内部，连按三下快捷键`F7`就会出现`sub_466020`的代码同前（为了便于阅读这里再抄写一遍）：

```
.text:00466020 sub_466020     proc near                   ; CODE XREF: sub_403EE0+14Dp

.text:00466020
.text:00466020 var_11C         = dword ptr -11Ch
.text:00466020 var_108         = dword ptr -108h

.text:00466020                push    ebx
.text:00466021                push    esi
.text:00466022                mov     esi, eax
.text:00466024                push    edi
.text:00466025                add     esp, 0FFFFFEFCh
.text:0046602B                xor     ebx, ebx
.text:0046602D                mov     eax, [esi+4]
.text:00466030                mov     edi, edx
.text:00466032                test    eax, eax
.text:00466034                jz      short loc_4660A0
.text:00466036                push    offset a_chm    ; ".chm"
.text:0046603B                mov     edx, ds:@Forms@Application ; Forms::Application
.text:00466041                mov     eax, [edx]
.text:00466043                add     eax, 50h
.text:00466046                mov     edx, [eax]
.text:00466048                test    edx, edx
.text:0046604A                jz      short loc_466050
.text:0046604C                mov     ecx, [eax]
.text:0046604E                jmp     short loc_466055
.text:00466050 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:00466050

.text:00466050 loc_466050:                      ; CODE XREF: sub_466020+2Aj
.text:00466050                mov     ecx, offset unk_555AF8

.text:00466055

.text:00466055 loc_466055:                      ; CODE XREF: sub_466020+2Ej
.text:00466055                push    ecx
.text:00466056                push    104h
.text:0046605B                lea     eax, [esp+114h+var_108]
.text:0046605F                push    eax
.text:00466060                call    IDA_1016
.text:00466065                push    esp
.text:00466066                call    IDA_992
.text:0046606B                test    al, al
.text:0046606D                jz      short loc_4660A0
.text:0046606F                test    edi, edi
.text:00466071                jnz     short loc_46608B
.text:00466073                push    0
.text:00466075                push    1
.text:00466077                lea     edx, [esp+124h+var_11C]
.text:0046607B                push    edx
.text:0046607C                push    0
.text:0046607E                call    dword ptr [esi+4]
.text:00466081                test    eax, eax
.text:00466083                setnz   bl
.text:00466086                and     ebx, 1
.text:00466089                jmp     short loc_4660A0
.text:0046608B ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0046608B

.text:0046608B loc_46608B:                      ; CODE XREF: sub_466020+51j
.text:0046608B                push    edi
.text:0046608C                push    0Fh
.text:0046608E                lea     eax, [esp+124h+var_11C]
.text:00466092                push    eax
.text:00466093                push    0
.text:00466095                call    dword ptr [esi+4]
.text:00466098                test    eax, eax
.text:0046609A                setnz   bl
.text:0046609D                and     ebx, 1

.text:004660A0

.text:004660A0 loc_4660A0:                      ; CODE XREF: sub_466020+14j
.text:004660A0                                      ; sub_466020+4Dj ...
.text:004660A0                mov     eax, ebx
.text:004660A2                add     esp, 104h
.text:004660A8                pop     edi
.text:004660A9                pop     esi
.text:004660AA                pop     ebx
.text:004660AB                retn
.text:004660AB sub_466020      endp
```

简言之在`.text:0046607B`处下断点（`F2`）或将光标定位在`.text:0046607B`（按快捷键`G`出现`Jump to address`对话框，输入`0046607B`后点击`OK`）处按快捷键`F4`，此时双击寄存器`edx`可以发现它指向帮助文件`idahelp.hlp`（我将`ida Freeware Vesion 4.9`安装在文件夹`C:\ida49free\`）：

```
Stack[00000B2C]:0013F460 aFTemp49freeApp db 'C:\ida49free\idahelp.chm',0
```

按快捷键`Ctrl+F2`结束程序调试。

有了参数就可以修改函数`sub_406640`了，将光标定位到如下一行代码（按快捷键`G`出现`Jump to address`对话框，输入`406640`后点击`OK`）：

```
.text:00406640                mov     eax, [eax+158h]
```

点击菜单`Edit->Patch program->Assemble...`出现`Assemble instruction`对话框，进行如下操作：

输入`mov eax, 57F4C0h`点击`OK`;

输入`xor edx, edx`点击`OK`;

输入`xor edx, edx`点击`OK`;

输入`xor edx, edx`点击`OK`;

输入`call 466020h`点击`OK`;

输入`ret`点击`OK`;

点击`Cancel`。

操作完成后得到如下代码：

```
.text:00406640 unk_406640      db 0B8h ; ?            ; DATA XREF: .data:0053AE9Fo
.text:00406641                db 0C0h ; ?
.text:00406642                db 0F4h ; ?
.text:00406643                db  57h ; W
.text:00406644                db    0
.text:00406645 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:00406645                xor     edx, edx
.text:00406647                xor     edx, edx
.text:00406649                xor     edx, edx
.text:0040664B                call    sub_466020
.text:00406650                retn
.text:00406651 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:00406651                add     edx, 50h
......
```

在`.text:00406640`行按快捷键`C`（Code 命令）及快捷键`P`（Program 命令）得到如下代码：

```
.text:00406640 sub_406640:     proc    near            ; DATA XREF: .data:0053AE9Fo
.text:00406640                mov     eax, offset off_57F4C0
.text:00406645                xor     edx, edx
.text:00406647                xor     edx, edx
.text:00406649                xor     edx, edx
.text:0040664B                call    sub_466020
.text:00406650                retn
.text:00406651 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:00406651                add     edx, 50h
.text:00406654                mov     eax, [edx]
.text:00406656                test    eax, eax
.text:00406658                jz      short loc_40665E
.text:0040665A                mov     ecx, [edx]
.text:0040665C                jmp     short loc_406663
.text:0040665E ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0040665E

.text:0040665E loc_40665E:                      ; CODE XREF: .text:00406658j
.text:0040665E                mov     ecx, offset szHelp

.text:00406663

.text:00406663 loc_406663:                      ; CODE XREF: .text:0040665Cj
.text:00406663                push    ecx         ; lpszHelp
.text:00406664                push    0           ; hWndMain
.text:00406666                call    WinHelpA
.text:0040666B                retn
```

到此为止补丁就打好了。

至于为何有三行`xor edx, edx`指令，那是因为在`.text:00406648`处有一个外部引用。在`.text:00406647`处按快捷键`U`(Undefine 命令)得到以下内容：

```
.text:00406640 sub_406640      proc near                   ; DATA XREF: .data:0053AE9Fo
.text:00406640                mov     eax, offset off_57F4C0
.text:00406645                xor     edx, edx
.text:00406645 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
```
