
```
004E73D5   |.  8BEC            mov ebp,esp                  ;很明显,004E750D处就是解密调用!

004E73D7   |.  83C4 F8         add esp,-8

..............

004E7414   |.  85C0            test eax,eax

004E7416   |.  7E 3F           jle short winchm.004E7457    ;  加密和还原

004E7418   |>  0FB7D7          /movzx edx,di                ;  05Fe:

004E741B   |.  C1EA 08         |shr edx,8

004E741E   |.  3213            |xor dl,[byte ds:ebx]        ;  与5 XOR

004E7420   |.  8816            |mov [byte ds:esi],dl

004E7422   |.  807D FB 00      |cmp [byte ss:ebp-5],0       ;  这是加解密的标志:1,加密;0,解密

004E7426   |.  74 17           |je short winchm.004E743F

004E7428   |.  81E2 FF000000   |and edx,0FF                 ;  XOR的结果只要未2位

004E742E   |.  66:03FA         |add di,dx                   ;  与初始码相加

004E7431   |.  66:69D7 540D    |imul dx,di,0D54             ;  dx==di\*0d54(结果保留word大小)

004E7436   |.  66:81C2 3422    |add dx,2234                 ;  +$2234

004E743B   |.  8BFA            |mov edi,edx                 ;  结果为下次XOR的密码

004E743D   |.  EB 13           |jmp short winchm.004E7452

004E743F   |>  33D2            |xor edx,edx                 ;  这是解密!!

004E7441   |.  8A13            |mov dl,[byte ds:ebx]

004E7443   |.  66:03FA         |add di,dx

004E7446   |.  66:69D7 540D    |imul dx,di,0D54

004E744B   |.  66:81C2 3422    |add dx,2234

004E7450   |.  8BFA            |mov edi,edx

004E7452   |>  46              |inc esi

004E7453   |.  43              |inc ebx

004E7454   |.  48              |dec eax

004E7455   |.^ 75 C1           \jnz short winchm.004E7418

004E7457   |>  5F              pop edi
```

作者大概是先加密写进注册表,在读出来解密验证了!很明显,`004E750D`处就是解密调用!

我们到`004E750D`处看看,果不出所料:

```
004E74FF    .  E8 D0FDFFFF     call winchm.004E72D4

004E7504    .  8B45 EC         mov eax,[dword ss:ebp-14]    ; |

004E7507    .  33C9            xor ecx,ecx                  ; |

004E7509    .  66:BA FE05      mov dx,5FE                   ; |

004E750D    .  E8 C2FEFFFF     call winchm.004E73D4         ; \winchm.004E73D4
```

向下我们可以定位到如下关键处:

```
004E758C    .  E8 1BCEF1FF     call winchm.004043AC

004E7591    .  8B45 DC         mov eax,[dword ss:ebp-24]     ;  (ASCII "scxtb")

004E7594    .  8D4D E4         lea ecx,[dword ss:ebp-1C]

004E7597    .  5A              pop edx                       ;  (00ECA07C), ASCII "123456789"

004E7598    .  E8 1BF6FFFF     call winchm.004E6BB8          ;  关键!!======>进入!!

004E759D    .  8B55 E4         mov edx,[dword ss:ebp-1C]

004E75A0    .  A1 0C485100     mov eax,[dword ds:51480C]

004E75A5    .  E8 EAD3F1FF     call winchm.00404994

004E75AA    .  A1 0C485100     mov eax,[dword ds:51480C]

004E75AF    .  8B00            mov eax,[dword ds:eax]

004E75B1    .  33D2            xor edx,edx

004E75B3    .  E8 14D5F1FF     call winchm.00404ACC

004E75B8    .  75 16           jnz short winchm.004E75D0

004E75BA    .  A1 244A5100     mov eax,[dword ds:514A24]

004E75BF    .  BA 28764E00     mov edx,winchm.004E7628       ;  UNICODE " [Unregistered]"

004E75C4    .  E8 F7D3F1FF     call winchm.004049C0

004E75C9    .  A1 244A5100     mov eax,[dword ds:514A24]

004E75CE    .  EB 0A           jmp short winchm.004E75DA

004E75D0    >  A1 244A5100     mov eax,[dword ds:514A24]

004E75D5    .  E8 56D2F1FF     call winchm.00404830
```

=====>进入`call winchm.004E6BB8`:

```
004E6BB8   /$  55              push ebp

004E6BB9   |.  8BEC            mov ebp,esp

004E6BBB   |.  51              push ecx

004E6BBC   |.  B9 05000000     mov ecx,5

004E6BC1   |>  6A 00           /push 0

........

004E6BF8   |.  E8 A3D5F1FF     call winchm.004041A0

004E6BFD   |.  8B45 EC         mov eax,[dword ss:ebp-14]

004E6C00   |.  E8 DFD7F1FF     call winchm.004043E4          ;  取注册码长度

004E6C05   |.  8B15 444B5100   mov edx,[dword ds:514B44]     ;  winchm.00515E0C

004E6C0B   |.  C702 FFFFFFFF   mov [dword ds:edx],-1

004E6C11   |.  83F8 14         cmp eax,14                    ;  Switch (cases 14..22)

004E6C14   |.  0F85 23010000   jnz winchm.004E6D3D           ;  14----22

004E6C1A   |.  A1 444B5100     mov eax,[dword ds:514B44]     ;  Case 14 of switch 004E6C11

004E6C1F   |.  33D2            xor edx,edx

004E6C21   |.  8910            mov [dword ds:eax],edx

004E6C23   |.  8B45 EC         mov eax,[dword ss:ebp-14]

004E6C26   |.  E8 69F8FFFF     call winchm.004E6494          ;  关键!!======>进入!!

004E6C2B   |.  8945 F0         mov [dword ss:ebp-10],eax

004E6C2E   |.  8955 F4         mov [dword ss:ebp-C],edx
```

这里说一点就行了:

```
004E6C11   |.  83F8 14         cmp eax,14             =====>比较假注册码长度;

004E6C14   |.  0F85 23010000   jnz winchm.004E6D3D    =====>每次比较长度后的跳转就是很好的爆破点;
```

现在我们可以一些对注册码长度的要求了. 注册码长度如下5种(14;18;19;1D;22):

```
004E6C05   |.  8B15 444B5100      mov edx,dword ptr ds:[514B44]  ;  winchm.00515E0C

004E6C0B   |.  C702 FFFFFFFF      mov dword ptr ds:[edx],-1

004E6C11   |.  83F8 14            cmp eax,14                     ;  Switch (cases 14..22)

004E6C14   |.  0F85 23010000      jnz winchm.004E6D3D            ;  14----22

004E6C1A   |.  A1 444B5100        mov eax,dword ptr ds:[514B44]  ;  Case 14 of switch 004E6C11

.......

004E6D3D   |> \83F8 19            cmp eax,19

004E6D40   |.  0F85 70010000      jnz winchm.004E6EB6

004E6D46   |.  8D45 E8            lea eax,[local.6]              ;  Case 19 of switch 004E6C11

.........

004E6EB6   |> \83F8 1D            cmp eax,1D

004E6EB9   |.  0F85 12010000      jnz winchm.004E6FD1

004E6EBF   |.  8D45 E8            lea eax,[local.6]              ;  Case 1D of switch 004E6C11

.......

004E6FD1   |> \83F8 22            cmp eax,22

004E6FD4   |.  0F85 E1000000      jnz winchm.004E70BB

004E6FDA   |.  8D45 E8            lea eax,[local.6]              ;  Case 22 of switch 004E6C11

.......

004E70BB   |> \83F8 18            cmp eax,18

004E70BE   |.  75 72              jnz short winchm.004E7132

004E70C0   |.  8D45 E8            lea eax,[local.6]              ;  Case 18 of switch 004E6C11
```

我们来看下注册码长为20(14H)的情形:(`call 004E6494`为关键!)

======>进入!!跟进后可以看见下面的循环,就是计算注册码了(我已将注册码重填为:12345678901234567891):

```
004E64DF   |. /7E 39              jle short winchm.004E651A

004E64E1   |. |BB 01000000        mov ebx,1                    ;  计算注册码了!!

004E64E6   |> |8BC3               /mov eax,ebx                 ;  从1开始取循环数,以决定取注册码的位数

004E64E8   |. |99                 |cdq                         ;  把EAX中的字的符号扩展到EDX中去

004E64E9   |. |52                 |push edx                    ; /Arg2

004E64EA   |. |50                 |push eax                    ; |Arg1

004E64EB   |. |B8 03000000        |mov eax,3                   ; |

004E64F0   |. |E8 5BFFFFFF        |call winchm.004E6450        ; \winchm.004E6450======>进入!!

004E64F5   |. |52                 |push edx                    ;  3^n: n==循环数+1

004E64F6   |. |50                 |push eax

004E64F7   |. |8B45 EC            |mov eax,[local.5]           ;  注册码 (ASCII "12345678901234567891")

004E64FA   |. |8A4418 FF          |mov al,byte ptr ds:[eax+ebx>;  取1位注册码的ASCII

004E64FE   |. |25 FF000000        |and eax,0FF                 ;  与上面结果进行

004E6503   |. |33D2               |xor edx,edx

004E6505   |. |E8 02ECF1FF        |call winchm.0040510C        ;  int64乘法

004E650A   |. |0345 E0            |add eax,[local.8]           ;  加上次结果

004E650D   |. |1355 E4            |adc edx,[local.7]           ;  带进位加法

004E6510   |. |8945 E0            |mov [local.8],eax

004E6513   |. |8955 E4            |mov [local.7],edx

004E6516   |. |43                 |inc ebx                     ;  循环次数+1::注册码的下1位

004E6517   |. |4E                 |dec esi                     ;  控制器-1

004E6518   |.^|75 CC              \jnz short winchm.004E64E6

004E651A   |> \8B45 E0            mov eax,[local.8]

004E651D   |.  8945 F0            mov [local.4],eax

004E6520   |.  8B45 E4            mov eax,[local.7]

004E6523   |.  8945 F4            mov [local.3],eax
```

======>进入!!上面的`call winchm.004E6450`如下(实质上是计算3^n):

```
004E6458   |.  8BF0               mov esi,eax

004E645A   |.  8BC6               mov eax,esi                  ;  esi==eax==3
```
