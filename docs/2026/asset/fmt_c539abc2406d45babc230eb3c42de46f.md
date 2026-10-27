
```asm
:004A66FB 8D4C2408                lea ecx, dword ptr [esp+08]
:004A66FF 899EC0D59201            mov dword ptr [esi+0192D5C0], ebx  ====>让dword ptr [esi+0192D5C0]等于ebx，估计是0
:004A6705 C7442420FFFFFFFF        mov [esp+20], FFFFFFFF
:004A670D E8EE860600              call 0050EE00
:004A6712 5E                      pop esi
:004A6713 B801000000              mov eax, 00000001
:004A6718 5B                      pop ebx
:004A6719 8B4C2410                mov ecx, dword ptr [esp+10]
:004A671D 64890D00000000          mov dword ptr fs:[00000000], ecx
:004A6724 83C41C                  add esp, 0000001C
:004A6727 C3                      ret

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004A6688(C), :004A669F(C), :004A66B6(C), :004A66C9(C)
|          ====>跳到这里就是MST 2003(企业版)

:004A6728 C705B4EA4C0302000000    mov dword ptr [034CEAB4], 00000002      ====>让dword ptr [034CEAB4]等于2
:004A6732 8D4C2408                lea ecx, dword ptr [esp+08]
:004A6736 C786C0D5920101000000    mov dword ptr [esi+0192D5C0], 00000001  ====>让dword ptr [esi+0192D5C0]等于1
:004A6740 C7442420FFFFFFFF        mov [esp+20], FFFFFFFF
:004A6748 E8B3860600              call 0050EE00
:004A674D 5E                      pop esi
:004A674E B801000000              mov eax, 00000001
:004A6753 5B                      pop ebx
:004A6754 8B4C2410                mov ecx, dword ptr [esp+10]
:004A6758 64890D00000000          mov dword ptr fs:[00000000], ecx
:004A675F 83C41C                  add esp, 0000001C
:004A6762 C3                      ret

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004A6630(C), :004A6647(C), :004A6671(C), :004A66EF(C)
|           ====>跳到这里就是MST 2003(学习版)

:004A6763 8D4C2408                lea ecx, dword ptr [esp+08]  ====>什么不作，dword ptr [034CEAB4]，dword ptr [esi+0192D5C0]等于初始值，当然为0！！！
:004A6767 C7442420FFFFFFFF        mov [esp+20], FFFFFFFF
:004A676F E88C860600              call 0050EE00

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A65F4(C)
| ====>一跳到这里就玩完了

:004A6774 8B4C2418                mov ecx, dword ptr [esp+18]
:004A6778 5E                      pop esi
:004A6779 33C0                    xor eax, eax
:004A677B 5B                      pop ebx
:004A677C 64890D00000000          mov dword ptr fs:[00000000], ecx
:004A6783 83C41C                  add esp, 0000001C
:004A6786 C3                      ret

:004A6787 90                      nop
:004A6788 90                      nop
:004A6789 90                      nop
:004A678A 90                      nop
:004A678B 90                      nop
:004A678C 90                      nop
:004A678D 90                      nop
:004A678E 90                      nop
:004A678F 90                      nop

:004A6790 8BC1                    mov eax, ecx
:004A6792 8B4C2404                mov ecx, dword ptr [esp+04]
:004A6796 8B80C0D59201            mov eax, dword ptr [eax+0192D5C0]
:004A679C 8B11                    mov edx, dword ptr [ecx]
:004A679E 50                      push eax
:004A679F FF12                    call dword ptr [edx]
:004A67A1 C20400                  ret 0004

:004A67A4 90                      nop
:004A67A5 90                      nop
:004A67A6 90                      nop
:004A67A7 90                      nop
:004A67A8 90                      nop
:004A67A9 90                      nop
:004A67AA 90                      nop
:004A67AB 90                      nop
:004A67AC 90                      nop
:004A67AD 90                      nop
:004A67AE 90                      nop
:004A67AF 90                      nop

:004A67B0 56                      push esi
:004A67B1 8BF1                    mov esi, ecx
:004A67B3 E8A0970600              call 0050FF58
:004A67B8 A1B4EA4C03              mov eax, dword ptr [034CEAB4]
:004A67BD 85C0                    test eax, eax
:004A67BF 750D                    jne 004A67CE

* Possible StringData Ref from Data Obj ->"MST 2003(学习版)"   ====>！就是这，往下看！！！！！！
                                 |
:004A67C1 6870215700              push 00572170
:004A67C6 8D4E5C                  lea ecx, dword ptr [esi+5C]
:004A67C9 E8BB870600              call 0050EF89

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A67BF(C)
|
:004A67CE 833DB4EA4C0301          cmp dword ptr [034CEAB4], 00000001
:004A67D5 750D                    jne 004A67E4

* Possible StringData Ref from Data Obj ->"MST 2003(设计版)"
                                 |
:004A67D7 685C215700              push 0057215C
:004A67DC 8D4E5C                  lea ecx, dword ptr [esi+5C]
:004A67DF E8A5870600              call 0050EF89

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A67D5(C)
|
:004A67E4 833DB4EA4C0302          cmp dword ptr [034CEAB4], 00000002
:004A67EB 750D                    jne 004A67FA

* Possible StringData Ref from Data Obj ->"MST 2003(企业版)"
                                 |
:004A67ED 6848215700              push 00572148
:004A67F2 8D4E5C                  lea ecx, dword ptr [esi+5C]
:004A67F5 E88F870600              call 0050EF89

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A67EB(C)
|
:004A67FA 6A00                    push 00000000
:004A67FC 8BCE                    mov ecx, esi
:004A67FE E806700600              call 0050D809
:004A6803 B801000000              mov eax, 00000001
:004A6808 5E                      pop esi
:004A6809 C3                      ret
```

.....省略

以下是检测时间的`call`，我们看看

```asm
* Referenced by a CALL at Address:
|:004A605B
|
:004A6A00 83EC08                  sub esp, 00000008
:004A6A03 8D442404                lea eax, dword ptr [esp+04]
:004A6A07 50                      push eax
:004A6A08 E8EB2B0500              call 004F95F8  ====>获得系统时间的call
:004A6A0D 8B4C2408                mov ecx, dword ptr [esp+08]
:004A6A11 83C404                  add esp, 00000004
:004A6A14 894C2400                mov dword ptr [esp], ecx
:004A6A18 8D4C2400                lea ecx, dword ptr [esp]
:004A6A1C 6A00                    push 00000000
:004A6A1E E854280600              call 00509277
:004A6A23 6A00                    push 00000000
:004A6A25 8D4C2404                lea ecx, dword ptr [esp+04]
:004A6A29 E849280600              call 00509277
:004A6A2E 6A00                    push 00000000
:004A6A30 8D4C2404                lea ecx, dword ptr [esp+04]
:004A6A34 E83E280600              call 00509277
:004A6A39 8B4014                  mov eax, dword ptr [eax+14]
:004A6A3C 056C070000              add eax, 0000076C
:004A6A41 3DD4070000              cmp eax, 000007D4   ====>7D4十进制为2004
:004A6A46 7E14                    jle 004A6A5C  ====>不大于2004年跳下去
:004A6A48 6A00                    push 00000000
:004A6A4A 6A00                    push 00000000

* Possible StringData Ref from Data Obj ->"您的软件应该升级了.请到[www.mstcenter.com下载](http://www.mstcenter.com下载)!"
                                 |
:004A6A4C 6804225700              push 00572204
:004A6A51 E85A020700              call 00516CB0
:004A6A56 33C0                    xor eax, eax
:004A6A58 83C408                  add esp, 00000008
:004A6A5B C3                      ret

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A6A46(C)
|
:004A6A5C 3DD3070000              cmp eax, 000007D3  ====>7D3十进制为2003
:004A6A61 7E0E                    jle 004A6A71  ====>不大于2003年跳下去,我觉得这是作者的笔误，应该是大于跳，否则没意义了。
:004A6A63 6A00                    push 00000000
:004A6A65 6A00                    push 00000000

* Possible StringData Ref from Data Obj ->"您的软件应该升级了.请到[www.mstcenter.com下载](http://www.mstcenter.com下载)!"
                                 |
:004A6A67 6804225700              push 00572204
:004A6A6C E83F020700              call 00516CB0

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A6A61(C)
|
:004A6A71 B801000000              mov eax, 00000001  ====>成功标志，eax＝1
```
