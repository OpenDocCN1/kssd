
```text
:0042F849 884DE1                  mov byte ptr [ebp-1F], cl  
                                  ==>CL=[ebp-10]=33   叁
                                  ==>33 即：3 入 [ebp-1F]

:0042F84C 90                      nop  
:0042F84D 90                      nop  
:0042F84E 90                      nop  
:0042F84F 90                      nop  

:0042F850 8B4DDC                  mov ecx, dword ptr [ebp-24]  
:0042F853 83C164                  add ecx, 00000064  
:0042F856 E845FDFEFF              call 0041F5A0  
:0042F85B 50                      push eax  
                                  ==>EAX=13572468  试炼码

:0042F85C 8D55E0                  lea edx, dword ptr [ebp-20]  
                                  ==>[ebp-20]处的值是上面 贰、叁、壹 的值

:0042F85F 52                      push edx  
                                  ==>EDX=Z31128  真码！

* Reference To: `KERNEL32.lstrcmpA`, Ord:02FCh  
                                  |

:0042F860 FF1504F34500            Call dword ptr [0045F304]  
                                  ==>比较注册码！呵呵，`lstrcmpA` 这个断点好找呀。

:0042F866 85C0                    test eax, eax  
:0042F868 750B                    jne 0042F875  
:0042F86A 90                      nop  
:0042F86B 90                      nop  
:0042F86C 90                      nop  
:0042F86D 90                      nop  
:0042F86E C745F801000000          mov [ebp-08], 00000001  
                                  ==>注册标志位 置 1  OK！

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
| `:0042F5E9(C), :0042F5FC(C), :0042F610(C), :0042F69D(U), :0042F868(C)`  
                                  ==>直接跳过来就 OVER 了！

:0042F875 90                      nop  
:0042F876 90                      nop  
:0042F877 90                      nop  
:0042F878 90                      nop  
:0042F879 8B45F8                  mov eax, dword ptr [ebp-08]  
                                  ==>注册标志位值 入 EAX！

:0042F87C 5F                      pop edi  
:0042F87D 5E                      pop esi  
:0042F87E 5B                      pop ebx  
:0042F87F 8BE5                    mov esp, ebp  
:0042F881 5D                      pop ebp  
:0042F882 C3                      ret
```

### 【KeyMake 之{47th}内存注册机】

- 中断地址：`42F85F`
- 中断次数：`1`
- 第一字节：`52`
- 指令长度：`1`
- 内存方式：`EDX`

### 【注册信息保存】

```reg
REGEDIT4

[HKEY_CURRENT_USER\Software\RJ Software\ClockWise\Registration]  
"User Name"="fly"  
"Serial Number"="1234"  
"Registration Code"="Z31128"
```

### 【整理】

- `User Name`：`fly`
- `Serial Number`：`1234`
- `Registration`：`Z31128`

---
Cracked By 巢水工作坊——fly【OCN】  
2003-03-16 18:43:04 `4dd466d0e1e0cc123afbe504da2462ca`
