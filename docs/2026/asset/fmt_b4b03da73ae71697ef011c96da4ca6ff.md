
0040D003  |.  33C8                     xor ecx,eax                      ;esi为第2段正确的regcode，edi为第3段正确的regcode

0040D005  |.  3BF1                     cmp esi,ecx                      ;cmp第2段regcode

0040D007  |.  75 1C                    jnz short SixthSen.0040D025

0040D009  |.  8B0D E08E4300            mov ecx,dword ptr ds:[438EE0]

0040D00F  |.  33C8                     xor ecx,eax

0040D011  |.  3BF9                     cmp edi,ecx                      ;cmp第3段regcode

0040D013  |.  75 10                    jnz short SixthSen.0040D025

0040D015  |.  8B45 FC                  mov eax,dword ptr ss:[ebp-4]

0040D018  |.  05 55410000              add eax,4155

0040D01D  |.  C700 D8070000            mov dword ptr ds:[eax],7D8       ;得到7D8后成功注册

0040D023  |.  EB 0E                    jmp short SixthSen.0040D033

0040D025  |>  8B45 FC                  mov eax,dword ptr ss:[ebp-4]

0040D028  |.  05 55410000              add eax,4155

0040D02D  |.  C700 D5070000            mov dword ptr ds:[eax],7D5       ;得到7D5后注册失败

0040D033  |>  8B00                     mov eax,dword ptr ds:[eax]

0040D035  |.  5F                       pop edi

0040D036  |.  5E                       pop esi

0040D037  |.  5B                       pop ebx

0040D038  |.  C9                       leave

0040D039  \.  C3                       retn

### 修改方法

```
0040D003  |.  33C8                     xor ecx,eax                      ;esi为第2段正确的regcode，edi为第3段正确的regcode

0040D005     /E9 D4E90100              jmp SixthSen.0042B9DE            ;跳到42B9DE执行附加的指令

0040D00A     |90                       nop

0040D00B     |90                       nop

0040D00C     |90                       nop

0040D00D     |90                       nop

0040D00E     |90                       nop

0040D00F  |. |33C8                     xor ecx,eax                      ;继续执行原程序指令
```

### 以下为在程序结尾空白处写入的附加指令

```
0042B9DE      BE DEBC0A89              mov esi,890ABCDE                 ;将第2段正确的regcode改为890ABCDE

0042B9E3      BF 78563412              mov edi,12345678                 ;将第3段正确的regcode改为12345678

0042B9E8      3BF1                     cmp esi,ecx

0042B9EA    ^ 0F85 3516FEFF            jnz SixthSen.0040D025

0042B9F0      8B0D E08E4300            mov ecx,dword ptr ds:[438EE0]

0042B9F6    ^ E9 1416FEFF              jmp SixthSen.0040D00F
```

****************************************************************************************************

用`HEdit`打开`SixthSense.exe`，找到相应地址`EIP-400C00`，`CTRL+G`修改代码

完毕

另外，注册信息保存在：

`[HKEY_LOCAL_MACHINE\SOFTWARE\小飞侠.NET\QQ第六感\Setting]`

`"ID"=""`

`"SN"=hex:`

by DimRacker

03:21 2005-03-18

[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
