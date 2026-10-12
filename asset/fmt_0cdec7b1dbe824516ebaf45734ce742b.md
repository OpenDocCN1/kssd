
## 汇编代码分析与 Patch 说明

### 原始指令段

```
0093900D     018D 85293509           add dword ptr ss:[ebp+9352985],ecx  
00939013     8BBD E12B3509           mov edi,dword ptr ss:[ebp+9352BE1]  
00939019     8BB5 112A3509           mov esi,dword ptr ss:[ebp+9352A11]  
0093901F     F3:A4                   rep movs byte ptr es:[edi],byte ptr ds:[>  
00939021     8BB5 7D203509           mov esi,dword ptr ss:[ebp+935207D]  
00939027     AD                      lods dword ptr ds:[esi]  
00939028     C746 FC 00000000        mov dword ptr ds:[esi-4],0  
0093902F     C1C0 05                rol eax,5  
00939032     05 27171E29            add eax,291E1727  
00939037     0385 2D273509          add eax,dword ptr ss:[ebp+935272D]  
0093903D     8B8D E12B3509          mov ecx,dword ptr ss:[ebp+9352BE1]  
00939043     8908                    mov dword ptr ds:[eax],ecx--------------上面 Shift+F9 后中断在这里  
```

### Patch 操作指南

将上面的`path①`和`path②`修改后，用`HideOD`申请一段内存地址，我这里申请到的是`02CF0000`。

将`00939043`改成`jmp 02cf0000` ------------------`Patch③`

### 后续指令段

```
00939045     AD                      lods dword ptr ds:[esi]  
00939046     C746 FC 00000000        mov dword ptr ds:[esi-4],0  
0093904D     89B5 7D203509          mov dword ptr ss:[ebp+935207D],esi-------注意这行地址 A  
00939053     83F8 FF               cmp eax,-1  
00939056     0F85 20000000         jnz yhds.0093907C  
0093905C     813E DDDDDDDD         cmp dword ptr ds:[esi],DDDDDDDD  
00939062     0F85 14000000         jnz yhds.0093907C  
00939068     C706 00000000         mov dword ptr ds:[esi],0  
0093906E     83C6 04              add esi,4  
00939071     89B5 7D203509         mov dword ptr ss:[ebp+935207D],esi  
00939077   ^ E9 5CF7FFFF          jmp yhds.009387D8  
0093907C     C1C0 03             rol eax,3  
0093907F     0385 2D273509         add eax,dword ptr ss:[ebp+935272D]  
00939085     83BD 59063509 01      cmp dword ptr ss:[ebp+9350659],1  
0093908C     0F84 9D000000         je yhds.0093912F  
00939092     813E AAAAAAAA         cmp dword ptr ds:[esi],AAAAAAAA  
00939098     0F85 12000000         jnz yhds.009390B0  
0093909E     83C6 04              add esi,4  
009390A1     C746 FC 00000000      mov dword ptr ds:[esi-4],0  
009390A8     97                     xchg eax,edi  
009390A9     B0 E9                 mov al,0E9  
009390AB     E9 03000000          jmp yhds.009390B3  
009390B0     97                     xchg eax,edi  
009390B1     B0 E8                 mov al,0E8  
009390B3     50                     push eax  
009390B4     83BD 15083509 01      cmp dword ptr ss:[ebp+9350815],1  
009390BB     0F84 3E000000         je yhds.009390FF  
009390C1     B8 00010000          mov eax,100  
009390C6     83BD 4CE84409 00      cmp dword ptr ss:[ebp+944E84C],0  
009390CD     0F84 08000000         je yhds.009390DB  
009390D3     8D9D CDEE4309         lea ebx,dword ptr ss:[ebp+943EECD]  
009390D9     FFD3                    call ebx  
009390DB     803F 90                 cmp byte ptr ds:[edi],90  
009390DE     0F84 08000000         je yhds.009390EC  
009390E4     83C7 05              add edi,5  
009390E7     E9 43000000          jmp yhds.0093912F  
009390EC     83F8 50              cmp eax,50  
009390EF     0F82 0A000000         jb yhds.009390FF  
009390F5     B0 90                mov al,90  
009390F7     AA                     stos byte ptr es:[edi]  
009390F8     58                     pop eax  
009390F9     AA                     stos byte ptr es:[edi]  
009390FA     E9 24000000          jmp yhds.00939123-----------Patch④  jmp 02cf0014  
009390FF     58                     pop eax  
00939100     AA                     stos byte ptr es:[edi]  
00939101     807F FF E9            cmp byte ptr ds:[edi-1],0E9  
00939105     0F85 18000000         jnz yhds.00939123------------Patch⑤  jmp 02cf0036  
0093910B     83BD 4CE84409 00      cmp dword ptr ss:[ebp+944E84C],0--------注意这行地址 C  
00939112     0F84 08000000         je yhds.00939120  
00939118     8D9D 9DEE4309         lea ebx,dword ptr ss:[ebp+943EE9D]  
0093911E     FFD3                    call ebx  
00939120     8847 04                mov byte ptr ds:[edi+4],al------------Patch⑥ NOP    去掉加密  
00939123     8B85 E12B3509         mov eax,dword ptr ss:[ebp+9352BE1]---------注意这行的地址 B  
00939129     2BC7                    sub eax,edi  
0093912B     83E8 04               sub eax,4  
0093912E     AB                     stos dword ptr es:[edi]------Patch⑦  NOP   去掉加密  
0093912F     AD                     lods dword ptr ds:[esi]  
00939130     C746 FC 00000000       mov dword ptr ds:[esi-4],0  
00939137   ^ E9 11FFFFFF          jmp yhds.0093904D-----循环处理每个 DLL 的函数  
                                      Patch⑧改为 jmp 02cf005F  
0093913C     89B5 7D203509         mov dword ptr ss:[ebp+935207D],esi  
00939142     52                     push edx  
00939143     68 00800000           push 8000  
00939148     6A 00                   push 0  
0093914A     FFB5 11283509         push dword ptr ss:[ebp+9352811]  
00939150     FF95 FD283509         call dword ptr ss:[ebp+93528FD]  
00939156     5A                     pop edx  
00939157     8B8D 95003509         mov ecx,dword ptr ss:[ebp+9350095]  
0093915D     C701 00000000         mov dword ptr ds:[ecx],0  
00939163     83C1 04               add ecx,4  
00939166     898D 95003509         mov dword ptr ss:[ebp+9350095],ecx  
0093916C   ^ E9 86F5FFFF          jmp yhds.009386F7  
00939171     E9 4B060000          jmp yhds.009397C1----这里 F2 下断，输入表处理完成后中断在这里  
```

### 内存写入说明

在`00939171`处`F2`下断后，`CTRL+G:02CF0000`到`02CF0000`写入`path`代码：(FLY 大侠的代码)

```
02CF0000     A3 0004CF02             mov dword ptr ds:[2CF0400],eax  
02CF0005     8908                    mov dword ptr ds:[eax],ecx  
02CF0007     AD                      lods dword ptr ds:[esi]  
02CF0008     C746 FC 00000000        mov dword ptr ds:[esi-4],0  
02CF000F   - E9 3990C4FD           jmp yhds.0093904D---------------------地址 A  
02CF0014     50                      push eax  
02CF0015     A1 0004CF02            mov eax,dword ptr ds:[2CF0400]  
02CF001A     8907                   mov dword ptr ds:[edi],eax  
02CF001C     807F FF E8             cmp byte ptr ds:[edi-1],0E8  
02CF0020     75 08                  jnz short 02CF002A  
02CF0022     66:C747 FE FF15        mov word ptr ds:[edi-2],15FF  
02CF0028     EB 06                  jmp short 02CF0030  
02CF002A     66:C747 FE FF25        mov word ptr ds:[edi-2],25FF  
02CF0030     58                      pop eax  
02CF0031   - E9 ED90C4FD           jmp yhds.00939123---------------------地址 B  
02CF0036     50                      push eax  
02CF0037     A1 0004CF02            mov eax,dword ptr ds:[2CF0400]  
02CF003C     8947 01                mov dword ptr ds:[edi+1],eax  
02CF003F     807F FF E8             cmp byte ptr ds:[edi-1],0E8  
02CF0043     75 08                  jnz short 02CF004D  
02CF0045     66:C747 FF FF15        mov word ptr ds:[edi-1],15FF  
02CF004B     EB 06                  jmp short 02CF0053  
02CF004D     66:C747 FF FF25        mov word ptr ds:[edi-1],25FF  
02CF0053     58                      pop eax  
02CF0054   - 0F85 C990C4FD         jnz yhds.00939123---------------------地址 B  
02CF005A   - E9 AC90C4FD           jmp yhds.0093910B---------------------地址 C  
02CF005F     83C7 04                add edi,4  
02CF0062   - E9 E68FC4FD           jmp yhds.0093904D---------------------地址 A  
02CF0067     90                      nop  
```

### 二进制代码

```
A3 00 04 CF 02 89 08 AD C7 46 FC 00 00 00 00 E9 39 90 C4 FD 50 A1 00 04 CF 02 89 07 80 7F FF E8
75 08 66 C7 47 FE FF 15 EB 06 66 C7 47 FE FF 25 58 E9 ED 90 C4 FD 50 A1 00 04 CF 02 89 47 01 80
```
