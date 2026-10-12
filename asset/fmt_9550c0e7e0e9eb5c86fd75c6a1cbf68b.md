
# chm 帮助编辑器 v2.6

> 原文：[`www.pediy.com/kssd/pediy03/forum568.htm`](https://www.pediy.com/kssd/pediy03/forum568.htm)

**目标软件**: `chm` 帮助编辑器 `v2.6`  
**破 解 人**: `TAE!`  
**保护方式**: 序列号  
**破解方式**: 分析出注册码  

**软件介绍**: 此软件的作者似乎处境相当困难，此篇的目的在于学习，请支持一下其作者，注册吧！  
**下载地址**: `http://kshii.51.net/`  

**序言**: 其实这个软件的注册码计算，比较部分相当简单，关键在于破解方法！如果用常规的方法即输入注册码，设断点，按注册，那就会什么反应都没有，那就换个方法吧！

软件的试用次数过了以后，会跳出一个 `Nag`，说已经过期，想想看，软件怎么知道我们有没有注册呢？肯定是读取了某处的一个注册标志！就从这里入手，如何？

在软件弹出 `Nag` 时，到 `TRW` 下命令 `Pmodule`，这时回到程序，按确定，又回到了 `TRW`（`Pmodule` 真是个很好用的命令），然后就向上查看，发现有两个地方可以跳过 `Nag`：

```
:0047A276 E8BDFDFFFF                  call 0047A038        //\*很关键的地方\*
:0047A27B A1E0B84800                  mov eax, dword ptr [0048B8E0]
:0047A280 8B80CC060000                mov eax, dword ptr [eax+000006CC]
:0047A286 81780CF4010000              cmp dword ptr [eax+0C], 000001F4
:0047A28D 0F845A010000                je 0047A3ED           //检查是否已经注册.(第一个)
:0047A293 6800010000                  push 00000100
:0047A298 8D85F7FEFFFF                lea eax, dword ptr [ebp+FFFFFEF7]
:0047A29E 50                          push eax
\* Possible StringData Ref from Data Obj ->""
                                        |
:0047A29F 6828A44700                  push 0047A428
\* Possible StringData Ref from Data Obj ->"PWes"          //win.ini 中的字符
                                        |
:0047A2A4 68F8A44700                  push 0047A4F8
\* Possible StringData Ref from Data Obj ->"AgeSet"        //win.ini 中的字符
                                        |
:0047A2A9 6834A44700                  push 0047A434
:0047A2AE E8E5C7F8FF                  call 00406A98
:0047A2B3 8D45FC                      lea eax, dword ptr [ebp-04]
:0047A2B6 8D95F7FEFFFF                lea edx, dword ptr [ebp+FFFFFEF7]
:0047A2BC B901010000                  mov ecx, 00000101
:0047A2C1 E85E9BF8FF                  call 00403E24
:0047A2C6 8D45F8                      lea eax, dword ptr [ebp-08]
:0047A2C9 50                          push eax
:0047A2CA B901000000                  mov ecx, 00000001
:0047A2CF BA01000000                  mov edx, 00000001
:0047A2D4 8B45FC                      mov eax, dword ptr [ebp-04]
:0047A2D7 E8A09DF8FF                  call 0040407C
\* Possible StringData Ref from Data Obj ->"1234567890!@#$%^-\*()QWERTYUIOPASDFGHJKLZXCVBNM"
                                        ->"qwertyuiopasdfghjklzxcvbnm"
                                        |
:0047A2DC BA44A44700                  mov edx, 0047A444
:0047A2E1 8B45F8                      mov eax, dword ptr [ebp-08]
:0047A2E4 E8779EF8FF                  call 00404160
:0047A2E9 8BD8                        mov ebx, eax
:0047A2EB 83FB3E                      cmp ebx, 0000003E
:0047A2EE 7E5D                        jle 0047A34D        //检查使用次数(第二个)
:0047A2F0 6A30                        push 00000030
\* Possible StringData Ref from Data Obj ->"软件过期"
                                        |
:0047A2F2 6808A54700                  push 0047A508
\* Possible StringData Ref from Data Obj ->"您的试用期限已过,若有继续使用本软件,请您注册."
                                        |
:0047A2F7 6814A54700                  push 0047A514
```

进入那个 `call 0047A038`

```
:0047A038 53                          push ebx
:0047A039 8BD8                        mov ebx, eax
:0047A03B 8B83BC060000                mov eax, dword ptr [ebx+000006BC]
:0047A041 8B4070                      mov eax, dword ptr [eax+70]
:0047A044 E80BFEFFFF                  call 00479E54       //\*计算比较注册码\*
:0047A049 84C0                        test al, al
:0047A04B 743A                        je 0047A087
:0047A04D 8B83BC060000                mov eax, dword ptr [ebx+000006BC]
:0047A053 8B4070                      mov eax, dword ptr [eax+70]
:0047A056 E8DD9FF8FF                  call 00404038
:0047A05B 50                          push eax
```

进入 `CAll 00479E54`

```
:00479E54 55                          push ebp
:00479E55 8BEC                        mov ebp, esp
:00479E57 33C9                        xor ecx, ecx
:00479E59 51                          push ecx
:00479E5A 51                          push ecx
:00479E5B 51                          push ecx
:00479E5C 51                          push ecx
:00479E5D 51                          push ecx
:00479E5E 51                          push ecx
:00479E5F 51                          push ecx
:00479E60 51                          push ecx
:00479E61 53                          push ebx
:00479E62 56                          push esi
:00479E63 8945FC                      mov dword ptr [ebp-04], eax
:00479E66 8B45FC                      mov eax, dword ptr [ebp-04]
:00479E69 E8BAA1F8FF                  call 00404028
:00479E6E 33C0                        xor eax, eax
:00479E70 55                          push ebp
\* Possible StringData Ref from Data Obj ->"闁�朊[迕"
                                        |
:00479E71 68EC9F4700                  push 00479FEC
:00479E76 64FF30                      push dword ptr fs:[eax]
:00479E79 648920                      mov dword ptr fs:[eax], esp
:00479E7C 33DB                        xor ebx, ebx
:00479E7E 8B45FC                      mov eax, dword ptr [ebp-04]
:00479E81 E8EE9FF8FF                  call 00403E74         //检查 Win.ini 中是否有一个 Pset=XXX(存放注册码的地方)
:00479E86 83F80D                      cmp eax, 0000000D     //比较注册码的位数是否为 13 位,我们在这里设断点.
:00479E89 0F8542010000                jne 00479FD1          //发现 EAX 不是 13 个跳走,是 13 个继续检查
```

所以在 `win.ini` 中加入这样一句 `Pset=1234567890abc`，在此运行程序此时被中断，继续 `F10`。

走到这里：

```
\* Possible StringData Ref from Data Obj ->"Ee"
                                        |
:00479EA8 BA04A04700                  mov edx, 0047A004
:00479EAD E8D2A0F8FF                  call 00403F84         //检查第一个和第二个字符是不是 Ee
:00479EB2 0F8519010000                jne 00479FD1          //不是就跳
:00479EB8 8D45F0                      lea eax, dword ptr [ebp-10]
:00479EBB 50                          push eax
:00479EBC B901000000                  mov ecx, 00000001
:00479EC1 BA03000000                  mov edx, 00000003
:00479EC6 8B45FC                      mov eax, dword ptr [ebp-04]
:00479EC9 E8AEA1F8FF                  call 0040407C
:00479ECE 8B45F0                      mov eax, dword ptr [ebp-10]
\* Possible StringData Ref from Data Obj ->"@"
                                        |
:00479ED1 BA10A04700                  mov edx, 0047A010    
:00479ED6 E8A9A0F8FF                  call 00403F84         //检查第三个字符是不是@
:00479EDB 0F85F0000000                 jne 00479FD1           //不是就跳
:00479EE1 8D45EC                      lea eax, dword ptr [ebp-14]
:00479EE4 50                          push eax
:00479EE5 B901000000                  mov ecx, 00000001
:00479EEA BA0A000000                  mov edx, 0000000A
:00479EEF 8B45FC                      mov eax, dword ptr [ebp-04]
:00479EF2 E885A1F8FF                  call 0040407C
:00479EF7 8B45EC                      mov eax, dword ptr [ebp-14]
\* Possible StringData Ref from Data Obj ->"%"
                                        |
:00479EFA BA1CA04700                  mov edx, 0047A01C
:00479EFF E880A0F8FF                  call 00403F84           //检查倒数第四个字符是不是%
:00479F04 0F85C7000000                jne 00479FD1
:00479F0A 8D45E8                      lea eax, dword ptr [ebp-18]
:00479F0D 50                          push eax
:00479F0E B901000000                  mov ecx, 00000001
:00479F13 BA0B000000                  mov edx, 0000000B
:00479F18 8B45FC                      mov eax, dword ptr [ebp-04]
:00479F1B E85CA1F8FF                  call 0040407C
:00479F20 8B45E8                      mov eax, dword ptr [ebp-18]
\* Possible StringData Ref from Data Obj ->"("
                                        |
:00479F23 BA28A04700                  mov edx, 0047A028
:00479F28 E857A0F8FF                  call 00403F84           //检查倒数第三个字符是不是(
:00479F2D 0F859E000000                jne 00479FD1
:00479F33 8D45E4                      lea eax, dword ptr [ebp-1C]
```
