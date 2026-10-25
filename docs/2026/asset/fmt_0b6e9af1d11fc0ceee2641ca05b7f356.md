
```asm
.text:0041CE5D                add eax, 4
.text:0041CE60                or      ecx, edx
.text:0041CE62                mov     edx, [esp+0E4h+var_BC+1]
.text:0041CE66                shl     ecx, 8
.text:0041CE69                and     edx, 0FFh
.text:0041CE6F                push    4
.text:0041CE71                or      ecx, edx
.text:0041CE73                mov     dword_46AE68, eax
.text:0041CE78                push    ecx
.text:0041CE79                push    eax
.text:0041CE7A                call    sub_41D9B0
.text:0041CE7F                mov     ecx, [esp+0F0h+var_C8+2]
.text:0041CE83                mov     edx, [esp+0F0h+var_C0]
.text:0041CE87                mov     eax, dword_46AE68
.text:0041CE8C                and     ecx, 0FFh
.text:0041CE92                shl     ecx, 8
.text:0041CE95                and     edx, 0FFh
.text:0041CE9B                add     eax, 4
.text:0041CE9E                or      ecx, edx
.text:0041CEA0                mov     edx, [esp+0F0h+var_BC+2]
.text:0041CEA4                shl     ecx, 8
.text:0041CEA7                and     edx, 0FFh
.text:0041CEAD                push    4
.text:0041CEAF                or      ecx, edx
.text:0041CEB1                mov     dword_46AE68, eax
.text:0041CEB6                push    ecx
.text:0041CEB7                push    eax
.text:0041CEB8                call    sub_41D9B0
.text:0041CEBD                mov     ecx, [esp+0FCh+var_C8+3]
.text:0041CEC1                mov     edx, [esp+0FCh+var_C0+1]
.text:0041CEC5                mov     eax, dword_46AE68
.text:0041CECA                and     ecx, 0FFh
.text:0041CED0                shl     ecx, 8
.text:0041CED3                and     edx, 0FFh
.text:0041CED9                add     eax, 4
.text:0041CEDC                or      ecx, edx
.text:0041CEDE                mov     edx, [esp+0FCh+var_BC+3]
.text:0041CEE2                shl     ecx, 8
.text:0041CEE5                and     edx, 0FFh
.text:0041CEEB                push    4
.text:0041CEED                or      ecx, edx
.text:0041CEEF                mov     dword_46AE68, eax
.text:0041CEF4                push    ecx
.text:0041CEF5                push    eax
.text:0041CEF6                call    sub_41D9B0
.text:0041CEFB                mov     ecx, [esp+108h+var_C4]
.text:0041CEFF                mov     eax, dword_46AE68
.text:0041CF04                mov     edx, [esp+108h+var_C0+2]
.text:0041CF08                and     ecx, 0FFh
.text:0041CF0E                add     eax, 4
.text:0041CF11                push    4
.text:0041CF13                shl     ecx, 8
.text:0041CF16                mov     dword_46AE68, eax
.text:0041CF1B                and     edx, 0FFh
.text:0041CF21                or      ecx, edx
.text:0041CF23                mov     edx, [esp+10Ch+var_C4+1]
.text:0041CF27                shl     ecx, 8
.text:0041CF2A                and     edx, 0FFh
.text:0041CF30                or      ecx, edx
.text:0041CF32                push    ecx
.text:0041CF33                push    eax
.text:0041CF34                call    sub_41D9B0
.text:0041CF39                mov     ecx, [esp+114h+var_C0+3]
.text:0041CF3D                mov     eax, dword_46AE68
.text:0041CF42                and     ecx, 0FFh
.text:0041CF48                add     eax, 4
.text:0041CF4B                push    2
.text:0041CF4D                push    ecx
.text:0041CF4E                push    eax
.text:0041CF4F                mov     dword_46AE68, eax
.text:0041CF54                call    sub_41D9B0
.text:0041CF59                mov     eax, dword_46AE68
.text:0041CF5E                add     esp, 48h
.text:0041CF61                add     eax, 2
.text:0041CF64                pop     edi
.text:0041CF65                mov     dword_46AE68, eax
.text:0041CF6A                pop     esi
.text:0041CF6B                mov     byte ptr [eax], 0
.text:0041CF6E                pop     ebp
.text:0041CF6F                mov     eax, offset unk_46AE6C
```

算法第四步完成，结果为`Result4[22]`

//////////////////////////////////////////////////////////////////////

```asm
.text:0041CF74                pop     ebx
.text:0041CF75                add     esp, 0C8h
.text:0041CF7B                retn
.text:0041CF7B sub_41CAD0      endp
.text:0041CF7B
.text:0041CF7B
; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
```

/////////////////////////////////////////////////////////////////////////////////
=================================================================================
/////////////////////////////////////////////////////////////////////////////////

第五步：连接：`"$1$"+用户名(前8个字节，用户名长度小于8个字节则用整个用户名)+"$"+Result4[22]`

设结果为`Result5`

`Result5`参加下一轮运算

```asm
.text:004075E2                call    sub_41CAD0　　　 ; 算法CALL（第二轮）
```

这一轮用到了注册界面中的"特征码”,设结果为`Result6`

连接：`"$1$+特征码前8个字节+"$"+Result6`

`.text:00407634` 之后

最终结果转换

===================================================================

分析完成：）

=================================

inside Pandora's Box

\*\*\*\*\*\*\*\*\*\*

CrAcKeD BY alphakk/iPB

=================================
