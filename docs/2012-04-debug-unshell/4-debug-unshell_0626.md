
# 手动脱壳 Enigma Protector 过程

> 原文：[`www.pediy.com/kssd/pediy11/122198.html`](https://www.pediy.com/kssd/pediy11/122198.html)

【原创】手动脱壳 Enigma Protector 过程（申请邀请码）

昨天下载了一个 Enigma Protector 加壳的二次加密软件练手，结果成功了，把过程发上来分享。

```
00421C6D     55            push ebp                                     ; 典型 Enigma Protector 入口
00421C6E     8BEC          mov ebp,esp
00421C70     83C4 F0       add esp,-10
00421C73     B8 00104000   mov eax,练习目标.00401000
00421C78     E8 01000000   call 练习目标.00421C7E
00421C7D     9A 83C4108B E>call far 5DE5:8B10C483
00421C84   - E9 DBC61E00   jmp 练习目标.0060E364                         ; F4 到这里，F8。
00421C89     3AB6 9F2324D6 cmp dh,byte ptr ds:\[esi+D624239F\]
```

\--------------------------------------------------------------------------------------------------------

Enigma Protector 的壳，开始遇到 `jmp` 直接 `F4`，这样快点。

经过 N 个 `jmp` 来到这里。

```
0060E73C   ^\\E9 D9FFFFFF   jmp 练习目标.0060E71A
0060E741     68 00400000   push 4000                                    ; F4 直接到这里
0060E746     68 00B40C00   push 0CB400
0060E74B     56            push esi
0060E74C     FF95 B8A01A00 call dword ptr ss:\[ebp+1AA0B8\]
......
0060E7B9     0118          add dword ptr ds:\[eax\],ebx
0060E7BB     49            dec ecx
0060E7BC     49            dec ecx
0060E7BD     83C7 02       add edi,2
0060E7C0   ^ E9 C8FFFFFF   jmp 练习目标.0060E78D
0060E7C5     E9 04000000   jmp 练习目标.0060E7CE                         ; F4 直接到这里
......
0060E7E5     3010          xor byte ptr ds:\[eax\],dl
0060E7E7     40            inc eax
0060E7E8     49            dec ecx
0060E7E9   ^ 0F85 F6FFFFFF jnz 练习目标.0060E7E5
0060E7EF     E9 04000000   jmp 练习目标.0060E7F8                         ; F4 直接到这里
0060E7F4     D5 68         aad 68                                         ; 观察这里代码变化
0060E7F6     1E            push ds
0060E7F7     17            pop ss
......
0060E88A     8945 00       mov dword ptr ss:\[ebp\],eax
0060E88D     58            pop eax
0060E88E     81C5 04000000 add ebp,4                                    ; 注意下面的 2 个 jmp
0060E894   ^ E9 BFFFFFFF   jmp 练习目标.0060E858                         ; jmp 返回
0060E899     5D            pop ebp
0060E89A     81C7 14000000 add edi,14
0060E8A0   ^ E9 5FFFFFFF   jmp 练习目标.0060E804                         ; jmp 返回
......
0060E8C8     8B40 04       mov eax,dword ptr ds:\[eax+4\]
0060E8CB     90            nop
0060E8CC     56            push esi
0060E8CD     03C6          add eax,esi
0060E8CF     FFD0          call eax                                       ; 看到这个就快返回了
0060E8D1     61            popad
0060E8D2     8B40 00       mov eax,dword ptr ds:\[eax\]
......
0060E8DF     8B70 3C       mov esi,dword ptr ds:\[eax+3C\]
0060E8E2     96            xchg eax,esi
0060E8E3     8B8430 280000>mov eax,dword ptr ds:\[eax+esi+28\]
0060E8EA     03C6          add eax,esi
0060E8EC     50            push eax
0060E8ED     C3            retn                                         ; F4 到这里，F8。
```

\--------------------------------------------------------------------------------------------------------

```
006B18C4     55            push ebp                                     ; 返回到这里了
006B18C5     8BEC          mov ebp,esp
006B18C7     83C4 C4       add esp,-3C
006B18CA     B8 24156B00   mov eax,练习目标.006B1524
006B18CF     E8 0868F6FF   call 练习目标.006180DC
006B18D4     A1 7C9E6C00   mov eax,dword ptr ds:\[6C9E7C\]
006B18D9     C600 01       mov byte ptr ds:\[eax\],1
006B18DC     E8 0B47F6FF   call 练习目标.00615FEC                         ; F4 到这里，F7
006B18E1     8D40 00       lea eax,dword ptr ds:\[eax\]
006B18E4     0000          add byte ptr ds:\[eax\],al
006B18E6     0000          add byte ptr ds:\[eax\],al
```

在这里首先要处理 IAT 加密，返回到这里后，`Ctrl+S`

搜索：`mov dword ptr ds:\[edx+eax*4+4\],ecx`

共找到 3 个，全部 `nop` 掉。

\=========================================================

```
00684494     8B0C8D B0856B>mov ecx,dword ptr ds:\[ecx*4+6B85B0\]
0068449B     894C82 04     mov dword ptr ds:\[edx+eax*4+4\],ecx          ; nop
0068449F     83C3 04       add ebx,4
006844A2     E9 D8000000   jmp 练习目标.0068457F
```

\=========================================================

```
00684592     8B0C8D 58866B>mov ecx,dword ptr ds:\[ecx*4+6B8658\]
00684599     894C82 04     mov dword ptr ds:\[edx+eax*4+4\],ecx          ; nop
0068459D     83C3 04       add ebx,4
006845A0     66:8B3B       mov di,word ptr ds:\[ebx\]
006845A3     83C3 02       add ebx,2
006845A6     66:85FF       test di,di
006845A9     74 5E         je short 练习目标.00684609
```

\=========================================================

```
00685171     8B4D F0       mov ecx,dword ptr ss:\[ebp-10\]
00685174     894C82 04     mov dword ptr ds:\[edx+eax*4+4\],ecx          ; nop
00685178     47            inc edi
00685179     FF4D D0       dec dword ptr ss:\[ebp-30\]
0068517C   ^ 0F85 CAFAFFFF jnz 练习目标.00684C4C
```

\-------------------------------------------------------------------------------------------------------

进入 `006B18DC` 的 `call` 里。

```
00615FEC     53            push ebx
00615FED     56            push esi
00615FEE     57            push edi
00615FEF     55            push ebp
00615FF0     BB 30B66C00   mov ebx,练习目标.006CB630
00615FF5     BE 00206B00   mov esi,练习目标.006B2000
00615FFA     BF 40B06C00   mov edi,练习目标.006CB040
00615FFF     807B 28 00    cmp byte ptr ds:\[ebx+28\],0
......
00615F30     31C0          xor eax,eax
00615F32     8705 00206B00 xchg dword ptr ds:\[6B2000\],eax
00615F38     F7D8          neg eax
00615F3A     19C0          sbb eax,eax
00615F3C     40            inc eax
00615F3D     BF 30B66C00   mov edi,练习目标.006CB630
00615F42     8B5F 18       mov ebx,dword ptr ds:\[edi+18\]
00615F45     8B6F 14       mov ebp,dword ptr ds:\[edi+14\]
00615F48     FF77 1C       push dword ptr ds:\[edi+1C\]
00615F4B     FF77 20       push dword ptr ds:\[edi+20\]
00615F4E     8B37          mov esi,dword ptr ds:\[edi\]
00615F50     B9 0B000000   mov ecx,0B
00615F55     F3:A5         rep movs dword ptr es:\[edi\],dword ptr d>
00615F57     5F            pop edi
00615F58     5E            pop esi
00615F59     C9            leave
00615F5A     C2 0C00       retn 0C                                      ; F4 到这里，F8 返回
00615F5D     C3            retn
```

\-------------------------------------------------------------------------------------------------------

返回到这里。

```
006B0A4F     C3            retn
006B0A50     55            push ebp                                      ; 练习目标.00400000
006B0A51     8BEC          mov ebp,esp
006B0A53     81C4 E4FEFFFF add esp,-11C
006B0A59     53            push ebx
006B0A5A     56            push esi
006B0A5B     57            push edi
006B0A5C     33C0          xor eax,eax
006B0A5E     8945 E8       mov dword ptr ss:\[ebp-18\],eax
006B0A61     8945 E4       mov dword ptr ss:\[ebp-1C\],eax
......
006B0AFA     8160 14 F00FF>and dword ptr ds:\[eax+14\],FFFF0FF0
006B0B01     8160 18 00DC0>and dword ptr ds:\[eax+18\],0DC00
006B0B08     C780 B8000000>mov dword ptr ds:\[eax+B8\],练习目标.006B0B15 ; 注意 6B0B15 地址
006B0B12     31C0          xor eax,eax
006B0B14     C3            retn
006B0B15     64:8F05 00000>pop dword ptr fs:\[0\]                         ; 6B0B15 在这里对应
006B0B1C     83C4 04       add esp,4
006B0B1F     68 3B0B6B00   push 练习目标.006B0B3B
......
006B0B61     8378 10 00    cmp dword ptr ds:\[eax+10\],0
006B0B65   ^ 0F85 B560FDFF jnz 练习目标.00686C20
006B0B6B     C780 B8000000>mov dword ptr ds:\[eax+B8\],练习目标.006B0B78 ; 注意 6B0B78 地址
006B0B75     31C0          xor eax,eax
006B0B77     C3            retn
006B0B78     64:8F05 00000>pop dword ptr fs:\[0\]                         ; 6B0B78 和这里对应
006B0B7F     83C4 04       add esp,4
006B0B82     B8 00000100   mov eax,10000                                  ; UNICODE
```



`"ALLUSERSPROFILE=C:\Documents and Settings\All Users"`  

......  

```
006B0BB9     B8 F70B6B00   mov eax,练习目标.006B0BF7  
006B0BBE     E8 85E6FCFF   call 练习目标.0067F248  
006B0BC3     010424        add dword ptr ss:[esp],eax  
006B0BC6     C3            retn                        ; F4 到这里，F8 返回
```

\--------------------------------------------------------------------------------------------------------  

返回到这里溜一圈。  

```
006B0BF7     C643 12 01    mov byte ptr ds:[ebx+12],1  
006B0BFB     E8 3CC6FCFF   call 练习目标.0067D23C  
006B0C00     E8 AB45FDFF   call 练习目标.006851B0  
006B0C05     C643 06 01    mov byte ptr ds:[ebx+6],1  
006B0C09     E8 FA34FDFF   call 练习目标.00684108
```

......  

```
006B0CD5     E8 AE77F6FF   call 练习目标.00618488          ; jmp to kernel32.GetComputerNameA  
006B0CDA     8B0D BC9F6C00 mov ecx,dword ptr ds:[6C9FBC]     ; 练习目标.006CBB14  
006B0CE0     8B15 509F6C00 mov edx,dword ptr ds:[6C9F50]     ; 练习目标.006CBA10  
006B0CE6     8B12          mov edx,dword ptr ds:[edx]  
006B0CE8     A1 889C6C00   mov eax,dword ptr ds:[6C9C88]  
006B0CED     E8 6631F7FF   call 练习目标.00623E58          ; 下面的注册表是个特征  
006B0CF2     68 C8146B00   push 练习目标.006B14C8          ; ASCII "Software\\Enigma Protector\\"  
006B0CF7     A1 A8A06C00   mov eax,dword ptr ds:[6CA0A8]
```

......  

```
006B0DFF     8160 14 F00FF>and dword ptr ds:[eax+14],FFFF0FF0  
006B0E06     8160 18 00DC0>and dword ptr ds:[eax+18],0DC00  
006B0E0D     C780 B8000000>mov dword ptr ds:[eax+B8],练习目标.006B0E1A ; 注意 6B0E1A 这个地址  
006B0E17     31C0          xor eax,eax  
006B0E19     C3            retn  
006B0E1A     64:8F05 00000>pop dword ptr fs:[0]                ; 6B0E1A 地址在这里对应  
006B0E21     83C4 04       add esp,4  
006B0E24     E8 0FE2FCFF   call 练习目标.0067F038  
006B0E29     E8 46E4FCFF   call 练习目标.0067F274  
006B0E2E     50            push eax  
006B0E2F     89C1          mov ecx,eax  
006B0E31     B8 6F0E6B00   mov eax,练习目标.006B0E6F  
006B0E36     E8 0DE4FCFF   call 练习目标.0067F248  
006B0E3B     010424        add dword ptr ss:[esp],eax  
006B0E3E     C3            retn                        ; F4 到这里，F8 返回
```

\--------------------------------------------------------------------------------------------------------  

返回后，`F8`单步走到返回。  

```
006B1095     E8 A2C1FCFF   call 练习目标.0067D23C  
006B109A     E8 D5E1FCFF   call 练习目标.0067F274  
006B109F     50            push eax  
006B10A0     89C1          mov ecx,eax  
006B10A2     B8 E0106B00   mov eax,练习目标.006B10E0  
006B10A7     E8 9CE1FCFF   call 练习目标.0067F248  
006B10AC     010424        add dword ptr ss:[esp],eax  
006B10AF     C3            retn                        ; F4 到这里，F8 返回
```

\-------------------------------------------------------------------------------------------------------  

返回到这里。  

```
006B10E0     68 FC106B00   push 练习目标.006B10FC  
006B10E5     64:FF35 00000>push dword ptr fs:[0]  
006B10EC     64:8925 00000>mov dword ptr fs:[0],esp  
006B10F3     31C0          xor eax,eax  
006B10F5     3100          xor dword ptr ds:[eax],eax  
006B10F7   ^ E9 245BFDFF   jmp 练习目标.00686C20
```

......  

```
006B1120     8160 14 F00FF>and dword ptr ds:[eax+14],FFFF0FF0  
006B1127     8160 18 00DC0>and dword ptr ds:[eax+18],0DC00  
006B112E     C780 B8000000>mov dword ptr ds:[eax+B8],练习目标.006B113B ; 注意 6B113B 这个地址  
006B1138     31C0          xor eax,eax  
006B113A     C3            retn  
006B113B     64:8F05 00000>pop dword ptr fs:[0]                ; 6B113B 地址在这里对应  
006B1142     83C4 04       add esp,4  
006B1145     A1 A8A06C00   mov eax,dword ptr ds:[6CA0A8]
```

......  

```
006B11E0     E8 9F60FDFF   call 练习目标.00687284  
006B11E5     A1 A8A06C00   mov eax,dword ptr ds:[6CA0A8]  
006B11EA     80B8 6A1C0000>cmp byte ptr ds:[eax+1C6A],0        ; F4 直接到这里  
006B11F1     74 0C         je short 练习目标.006B11FF          ; 修改[eax+1C6A]=0  
006B11F3     A1 A8A06C00   mov eax,dword ptr ds:[6CA0A8]  
006B11F8     8B00          mov eax,dword ptr ds:[eax]  
006B11FA     E8 F95BFDFF   call 练习目标.00686DF8
```

......  

```
006B12B7     E8 30F6FFFF   call 练习目标.006B08EC  
006B12BC     33C0          xor eax,eax  
006B12BE     5A            pop edx  
006B12BF     59            pop ecx  
006B12C0     59            pop ecx  
006B12C1     64:8910       mov dword ptr fs:[eax],edx  
006B12C4     EB 53         jmp short 练习目标.006B1319         ; F4 直接到这里
```

......  

```
006B131D     E8 CEBDFCFF   call 练习目标.0067D0F0  
006B1322     E8 4DDFFCFF   call 练习目标.0067F274  
006B1327     50            push eax  
006B1328     89C1          mov ecx,eax  
006B132A     B8 68136B00   mov eax,练习目标.006B1368  
006B132F     E8 14DFFCFF   call 练习目标.0067F248  
006B1334     010424        add dword ptr ss:[esp],eax  
006B1337     C3            retn                        ; F4 到这里，F8 返回
```

\--------------------------------------------------------------------------------------------------------  

返回到这里了。  

```
006B1368     A1 A4A06C00   mov eax,dword ptr ds:[6CA0A4]       ; 返回到这里  
006B136D     8B00          mov eax,dword ptr ds:[eax]  
006B136F     8B15 A8A06C00 mov edx,dword ptr ds:[6CA0A8]       ; 练习目标.006CBB50  
006B1375     0342 28       add eax,dword ptr ds:[edx+28]  
006B1378     8B15 A8A06C00 mov edx,dword ptr ds:[6CA0A8]       ; 练习目标.006CBB50  
006B137E     0382 EC000000 add eax,dword ptr ds:[edx+EC]  
006B1384     83C0 18       add eax,18
```

......  

```
006B1470     E8 03F0FCFF   call 练习目标.00680478  
006B1475     8B45 14       mov eax,dword ptr ss:[ebp+14]  
006B1478     50            push eax  
006B1479     A1 A8A06C00   mov eax,dword ptr ds:[6CA0A8]  
006B147E     8B80 0C010000 mov eax,dword ptr ds:[eax+10C]  
006B1484     50            push eax  
006B1485     8B45 18       mov eax,dword ptr ss:[ebp+18]  
006B1488     50            push eax                      ; 看上面的代码多熟悉呀  
006B1489     E8 0EF0FCFF   call 练习目标.0068049C        ; F4 到这里，F7 进入  
006B148E     E8 8D57FDFF   call 练习目标.00686C20
```

\--------------------------------------------------------------------------------------------------------  

进入 `006B1489` 的 `call` 里。  

```
0068049C     55            push ebp                    ; 固化了的格式，没问题  
0068049D     8BEC          mov ebp,esp  
0068049F     83C4 F4       add esp,-0C  
006804A2     53            push ebx  
006804A3     56            push esi  
006804A4     57            push edi  
006804A5     8B7D 0C       mov edi,dword ptr ss:[ebp+C]  
006804A8     8D5D F8       lea ebx,dword ptr ss:[ebp-8]  
006804AB     8D75 F4       lea esi,dword ptr ss:[ebp-C]  
006804AE     B8 00020000   mov eax,200  
006804B3     E8 3442F9FF   call 练习目标.006146EC
```

......  

向下拉 N 多的代码，到这里了。  

```
00680B80     E8 478BFAFF   call 练习目标.006296CC  
00680B85     8B45 FC       mov eax,dword ptr ss:[ebp-4]  
00680B88   - FFE0          jmp eax                     ; F4 直接到这里  
00680B8A     5F            pop edi  
00680B8B     5E            pop esi  
00680B8C     5B            pop ebx  
00680B8D     8BE5          mov esp,ebp  
00680B8F     5D            pop ebp  
00680B90     C2 0C00       retn 0C
```

\--------------------------------------------------------------------------------------------------------  

从 `00680B88` 跳来。  

```
015867A0     E8 CF8A0FFF   call 练习目标.0067F274          ; 来到这里，F8 单步  
015867A5     E9 04000000   jmp 015867AE  
015867AA     5A            pop edx  
015867AB     91            xchg eax,ecx  
015867AC   ^ 7D 82        jge short 01586730  
015867AE     8905 90376E00 mov dword ptr ds:[6E3790],eax
```

......  

省略 N 个 `jmp`  

```
015868A1   ^\\0F85 D8FFFFFF jnz 0158687F                  ; 别回跳。  
015868A7     E9 04000000   jmp 015868B0                  ; F4 直接到这里  
015868AC     F681 F05A81C4>test byte ptr ds:[ecx+C4815AF0],7C
```

......  



# 脱壳与OEP修正

## 汇编代码片段

```
省略 N 个 jmp

015B305C     68 34144000   push 401434             ; ASCII "VB5!6&vb6chs.dll"
015B3061     E8 2AE2E4FE   call 练习目标.00401290
015B3066   - E9 37E2E4FE   jmp 练习目标.004012A2
```

终于到地方了，原来 VB 的。

---

## 定位OEP

`Ctrl+G`: `00401000`
`Ctrl+B`: `FF 25`

找到这里了。

```
0040111C     90            nop
0040111D   - E9 AF1E1A00   jmp 练习目标.005A2FD1
00401122   - FF25 80104000 jmp dword ptr ds:[401080]          ; msvbvm60.rtcShell
00401128     90            nop
00401129   - E9 AC1E1A00   jmp 练习目标.005A2FDA
0040112E   - FF25 A0104000 jmp dword ptr ds:[4010A0]          ; msvbvm60.rtcStringBstr

......

00401279   - E9 581E1A00   jmp 练习目标.005A30D6
0040127E   - FF25 9C104000 jmp dword ptr ds:[40109C]          ; msvbvm60.GetMemNewObj
00401284     90            nop
00401285   - E9 551E1A00   jmp 练习目标.005A30DF
0040128A   - FF25 AC104000 jmp dword ptr ds:[4010AC]          ; msvbvm60.SetMemNewObj
00401290     90            nop
00401291   - E9 521E1A00   jmp 练习目标.005A30E8
00401296     0000          add byte ptr ds:[eax],al
00401298     E8 7E483E4D   call 4D7E5B1B
0040129D     B7 01         mov bh,1
0040129F     A4            movs byte ptr es:[edi],byte ptr ds:[esi>
```

## 修正OEP

按照 VB 的 OEP 格式应该在 `00401298`，复制 `015868A1` 二行代码，修正 OEP，新建 EIP。

```
00401285   - E9 551E1A00   jmp 练习目标.005A30DF
0040128A   - FF25 AC104000 jmp dword ptr ds:[4010AC]          ; msvbvm60.SetMemNewObj
00401290     90            nop
00401291   - E9 521E1A00   jmp 练习目标.005A30E8
00401296     0000          add byte ptr ds:[eax],al
00401298     68 34144000   push 练习目标.00401434          ; ASCII "VB5!6&vb6chs.dll"
0040129D     E8 EEFFFFFF   call 练习目标.00401290
004012A2     0000          add byte ptr ds:[eax],al
004012A4     40            inc eax
004012A5     0000          add byte ptr ds:[eax],al
```

---

## 保存与测试

使用 `LordPE` 转存为 `dumped.exe`。

使用 `ImportREC`，设置 `oep=401298-400000=00001298`，`rva=00001000`。

保存。

使用 `PEiD` 测试为 `Microsoft Visual Basic 5.0 / 6.0`。

至此，脱壳成功。

因属于商业软件，这里就不提供附件了。有兴趣的可以留言。
