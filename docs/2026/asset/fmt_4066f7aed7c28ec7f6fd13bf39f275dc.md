
0093907F     0385 2D273509           add eax,dword ptr ss:[ebp+935272D]

00939085     83BD 59063509 01        cmp dword ptr ss:[ebp+9350659],1

0093908C     0F84 9D000000           je yhds.0093912F

00939092     813E AAAAAAAA           cmp dword ptr ds:[esi],AAAAAAAA

00939098     0F85 12000000           jnz yhds.009390B0

0093909E     83C6 04                 add esi,4

009390A1     C746 FC 00000000        mov dword ptr ds:[esi-4],0

009390A8     97                      xchg eax,edi

009390A9     B0 E9                   mov al,0E9

009390AB     E9 03000000             jmp yhds.009390B3

009390B0     97                      xchg eax,edi

009390B1     B0 E8                   mov al,0E8

009390B3     50                      push eax

009390B4     83BD 15083509 01        cmp dword ptr ss:[ebp+9350815],1

009390BB     0F84 3E000000           je yhds.009390FF

009390C1     B8 00010000             mov eax,100

009390C6     83BD 4CE84409 00        cmp dword ptr ss:[ebp+944E84C],0

009390CD     0F84 08000000           je yhds.009390DB

009390D3     8D9D CDEE4309           lea ebx,dword ptr ss:[ebp+943EECD]

009390D9     FFD3                    call ebx

009390DB     803F 90                 cmp byte ptr ds:[edi],90

009390DE     0F84 08000000           je yhds.009390EC

009390E4     83C7 05                 add edi,5

009390E7     E9 43000000             jmp yhds.0093912F

009390EC     83F8 50                 cmp eax,50

009390EF     0F82 0A000000           jb yhds.009390FF

009390F5     B0 90                   mov al,90

009390F7     AA                      stos byte ptr es:[edi]

009390F8     58                      pop eax

009390F9     AA                      stos byte ptr es:[edi]

009390FA     E9 24000000             jmp yhds.00939123-----------Patch④  jmp 02cf0014

009390FF     58                      pop eax

00939100     AA                      stos byte ptr es:[edi]

00939101     807F FF E9              cmp byte ptr ds:[edi-1],0E9

00939105     0F85 18000000           jnz yhds.00939123------------Patch⑤  jmp 02cf0036

0093910B     83BD 4CE84409 00        cmp dword ptr ss:[ebp+944E84C],0--------注意这行地址C

00939112     0F84 08000000           je yhds.00939120

00939118     8D9D 9DEE4309           lea ebx,dword ptr ss:[ebp+943EE9D]

0093911E     FFD3                    call ebx

00939120     8847 04                 mov byte ptr ds:[edi+4],al------------Patch⑥ NOP    去掉加密

00939123     8B85 E12B3509           mov eax,dword ptr ss:[ebp+9352BE1]---------注意这行的地址B

00939129     2BC7                    sub eax,edi

0093912B     83E8 04                 sub eax,4

0093912E     AB                      stos dword ptr es:[edi]------Patch⑦  NOP   去掉加密

0093912F     AD                      lods dword ptr ds:[esi]

00939130     C746 FC 00000000        mov dword ptr ds:[esi-4],0

00939137   ^ E9 11FFFFFF             jmp yhds.0093904D-----循环处理每个DLL的函数

                                                        Patch⑧改为 jmp 02cf005F

0093913C     89B5 7D203509           mov dword ptr ss:[ebp+935207D],esi

00939142     52                      push edx

00939143     68 00800000             push 8000

00939148     6A 00                   push 0

0093914A     FFB5 11283509           push dword ptr ss:[ebp+9352811]

00939150     FF95 FD283509           call dword ptr ss:[ebp+93528FD]

00939156     5A                      pop edx

00939157     8B8D 95003509           mov ecx,dword ptr ss:[ebp+9350095]

0093915D     C701 00000000           mov dword ptr ds:[ecx],0

00939163     83C1 04                 add ecx,4

00939166     898D 95003509           mov dword ptr ss:[ebp+9350095],ecx

0093916C   ^ E9 86F5FFFF             jmp yhds.009386F7

00939171     E9 4B060000             jmp yhds.009397C1----这里F2下断，输入表处理完成后中断在这里

在`00939171`处F2下断后,CTRL+G:`02CF0000`到`02CF0000`写入path代码:  (FLY大侠的代码)

```
02CF0000     A3 0004CF02             mov dword ptr ds:[2CF0400],eax

02CF0005     8908                    mov dword ptr ds:[eax],ecx

02CF0007     AD                      lods dword ptr ds:[esi]

02CF0008     C746 FC 00000000        mov dword ptr ds:[esi-4],0

02CF000F   - E9 3990C4FD             jmp yhds.0093904D---------------------地址A

02CF0014     50                      push eax

02CF0015     A1 0004CF02             mov eax,dword ptr ds:[2CF0400]

02CF001A     8907                    mov dword ptr ds:[edi],eax

02CF001C     807F FF E8              cmp byte ptr ds:[edi-1],0E8

02CF0020     75 08                   jnz short 02CF002A

02CF0022     66:C747 FE FF15         mov word ptr ds:[edi-2],15FF

02CF0028     EB 06                   jmp short 02CF0030

02CF002A     66:C747 FE FF25         mov word ptr ds:[edi-2],25FF

02CF0030     58                      pop eax

02CF0031   - E9 ED90C4FD             jmp yhds.00939123---------------------地址B

02CF0036     50                      push eax

02CF0037     A1 0004CF02             mov eax,dword ptr ds:[2CF0400]

02CF003C     8947 01                 mov dword ptr ds:[edi+1],eax

02CF003F     807F FF E8              cmp byte ptr ds:[edi-1],0E8

02CF0043     75 08                   jnz short 02CF004D

02CF0045     66:C747 FF FF15         mov word ptr ds:[edi-1],15FF

02CF004B     EB 06                   jmp short 02CF0053

02CF004D     66:C747 FF FF25         mov word ptr ds:[edi-1],25FF

02CF0053     58                      pop eax

02CF0054   - 0F85 C990C4FD           jnz yhds.00939123---------------------地址B

02CF005A   - E9 AC90C4FD             jmp yhds.0093910B---------------------地址C

02CF005F     83C7 04                 add edi,4

02CF0062   - E9 E68FC4FD             jmp yhds.0093904D---------------------地址A

02CF0067     90                      nop
```

二进制代码:

```
A3 00 04 CF 02 89 08 AD C7 46 FC 00 00 00 00 E9 39 90 C4 FD 50 A1 00 04 CF 02 89 07 80 7F FF E8

75 08 66 C7 47 FE FF 15 EB 06 66 C7 47 FE FF 25 58 E9 ED 90 C4 FD 50 A1 00 04 CF 02 89 47 01 80

7F FF E8 75 08 66 C7 47 FF FF 15 EB 06 66 C7 47 FF FF 25 58 0F 85 C9 90 C4 FD E9 AC 90 C4 FD 83

C7 04 E9 E6 8F C4 FD 90
```

写好代码后,取消先前在code段下的内存断点,shift+F9,中断在`00939171`,到这里已经获得了IAT,现在找OEP.

在这里我按FLY大侠重开一个OD,shift+F9到程序运行后Ctrl+B在整个段块搜索Hex值：9D C3 E9找OEP的方法

没能成功,可能是我自己的操作问题,有成功经验的朋友希望介绍一下.

这里提供一个更快找themida保护程序OEP的方法,具体的原理是什么,我也不清楚,是一次偶然失误发现的(未必适

合所有程序)

在`00939171`处断下后,取消断点,ALT+M打开内存察看窗口，直接在代码段F2下断点。Shift+F9断在OEP处

```
0059F96C     55                                push ebp-----------------OEP

0059F96D     8BEC                              mov ebp,esp

0059F96F     83C4 F0                           add esp,-10

0059F972     53                                push ebx

0059F973     B8 7CF15900                       mov eax,yhds.0059F17C

0059F978     E8 8778E6FF                       call yhds.00407204

0059F97D     8B1D 183C5A00                     mov ebx,dword ptr ds:[5A3C18]            ; yhds.005A4C38

0059F983     68 60FC5900                       push yhds.0059FC60                       ; ASCII "yyyy-MM-dd"

0059F988     6A 1F                             push 1F

0059F98A     E8 157CE6FF                       call yhds.004075A4

0059F98F     50                                push eax

0059F990     E8 877DE6FF                       call yhds.0040771C

0059F995     68 6CFC5900                       push yhds.0059FC6C
```

到这里就可以用LordPE纠正大小后完整dump出来了,打开ImportREC F1.6, OEP:`19F96C`,自动搜索,获得有效指针

RAV:`001A61F0`, 大小:`9CC`,剪掉一个无效指针后修改dump出来的文件,修复后PEID0.94显示是Borland Delphi 6.0 - 7.0

编写的程序,修复后程序能正常运行.就是退出来时有出错提示(这个我就不知道怎么弄了,有请知道原因的朋友解

析一下).

--------------------------------------------------------------------------------

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
