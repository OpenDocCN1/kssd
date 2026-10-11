
## 算法跟踪记录

```
:00479DE8 8BC6                  mov eax, esi  
:00479DEA E83DE5FFFF           call 0047832C  
:00479DEF 8D9324030000         lea edx, dword ptr [ebx+00000324]  
:00479DF5 8BC6                  mov eax, esi  
:00479DF7 E830E5FFFF           call 0047832C  
:00479DFC 8D9328030000         lea edx, dword ptr [ebx+00000328]  
:00479E02 8BC6                  mov eax, esi  
:00479E04 E823E5FFFF           call 0047832C  
:00479E09 6A00                  push 00000000  
:00479E0B 8B9324030000         mov edx, dword ptr [ebx+00000324]  
:00479E11 8B8320030000         mov eax, dword ptr [ebx+00000320]
```

* Possible StringData Ref from Code Obj -> `~TurnIde@$IntoProfit$w/eBook$!>>YouC@n$ucceed!`
    -> `<<`
    |

```
:00479E17 B9789E4700           mov ecx, 00479E78  
:00479E1C E83BE2FFFF           call 0047805C      <=== 算法 CALL 了，最后一次 F8 进入，看个究竟  
:00479E21 84C0                  test al, al  
:00479E23 7544                  jne 00479E69      <=== 关键跳转(终于到了)，爆破只针对每个 exe 文件，所以没多大意思  
:00479E25 8BCB                  mov ecx, ebx  
:00479E27 B201                  mov dl, 01
```

* Possible StringData Ref from Code Obj -> `?C`
    |

```
:00479E29 A1B4744700           mov eax, dword ptr [004774B4]  
:00479E2E E83597FCFF           call 00443568  
:00479E33 8945FC               mov dword ptr [ebp\-04], eax  
:00479E36 33C0                 xor eax, eax  
:00479E38 55                   push ebp  
:00479E39 68629E4700           push 00479E62  
:00479E3E 64FF30               push dword ptr fs:[eax]  
:00479E41 648920               mov dword ptr fs:[eax], esp  
:00479E44 8B45FC               mov eax, dword ptr [ebp\-04]  
:00479E47 E88CD8FFFF           call 004776D8  
:00479E4C 33C0                 xor eax, eax  
:00479E4E 5A                   pop edx  
:00479E4F 59                   pop ecx  
:00479E50 59                   pop ecx  
:00479E51 648910               mov dword ptr fs:[eax], edx  
:00479E54 68699E4700           push 00479E69
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |
  | `:00479E67(U)`
  |

```
:00479E59 8B45FC               mov eax, dword ptr [ebp\-04]  
:00479E5C E85F90F8FF           call 00402EC0  
:00479E61 C3                    ret

:00479E62 E9B997F8FF           jmp 00403620  
:00479E67 EBF0                  jmp 00479E59
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |
  | `:00479E23(C)`
  |

```
:00479E69 5E                   pop esi  
:00479E6A 5B                   pop ebx  
:00479E6B 59                   pop ecx  
:00479E6C 5D                   pop ebp  
:00479E6D C3                    ret
```

## F8 来到下面代码段

```
:0047805C 55                   push ebp      <=== 此处下 D EAX 发现 EAX=newlaos，下 D EDX 发现 EDX＝78787878  
:0047805D 8BEC                 mov ebp, esp  
:0047805F 81C4FCFEFFFF         add esp, FFFFFEFC  
:00478065 53                   push ebx  
:00478066 56                   push esi  
:00478067 57                   push edi  
:00478068 33DB                 xor ebx, ebx  
:0047806A 895DFC               mov dword ptr [ebp\-04], ebx  
:0047806D 8BF9                 mov edi, ecx  
:0047806F 8BF2                 mov esi, edx  
:00478071 8BD8                 mov ebx, eax  
:00478073 8B4508               mov eax, dword ptr [ebp+08]  
:00478076 E85DBFF8FF           call 00403FD8  
:0047807B 33C0                 xor eax, eax  
:0047807D 55                   push ebp  
:0047807E 68D7804700           push 004780D7  
:00478083 64FF30               push dword ptr fs:[eax]  
:00478086 648920               mov dword ptr fs:[eax], esp  
:00478089 8D85FCFEFFFF         lea eax, dword ptr [ebp+FFFFFEFC]  
:0047808F 50                   push eax  
:00478090 8B4D08               mov ecx, dword ptr [ebp+08]  
:00478093 8BD7                 mov edx, edi  
:00478095 8BC3                 mov eax, ebx  
:00478097 E810FEFFFF           call 00477EAC   <=== 此 CALL 算出与用户名相对应的注册码  
:0047809C 8D95FCFEFFFF         lea edx, dword ptr [ebp+FFFFFEFC]  
:004780A2 8D45FC               lea eax, dword ptr [ebp\-04]  <=== 此处下 D EAX 发现注册码，但位置不正确  
:004780A5 E81EBDF8FF           call 00403DC8  
:004780AA 8B45FC               mov eax, dword ptr [ebp\-04]  <=== 此处将真码放入 EAX  
:004780AD 8BD6                 mov edx, esi     <=== 将假码 78787878 放入 EDX，在这下命令 D EAX 可以看见注册码  
:004780AF E880BEF8FF           call 00403F34    
:004780B4 0F94C0               sete al  
:004780B7 8BD8                 mov ebx, eax  
:004780B9 33C0                 xor eax, eax  
:004780BB 5A                   pop edx  
:004780BC 59                   pop ecx  
:004780BD 59                   pop ecx  
:004780BE 648910               mov dword ptr fs:[eax], edx  
:004780C1 68DE804700           push 004780DE
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |
  | `:004780DC(U)`
  |

```
:004780C6 8D45FC               lea eax, dword ptr [ebp\-04]  
:004780C9 E8D6BAF8FF           call 00403BA4  
:004780CE 8D4508               lea eax, dword ptr [ebp+08]  
:004780D1 E8CEBAF8FF           call 00403BA4  
:004780D6 C3                    ret
```

## 4、制作内存注册机 (keymake 1.73)

### 一、选择 F8 → 另类注册机！

*   程序名称：`crack.exe`
*   添加数据：
    *   中断地址：`004780AD`
    *   中断次数：`1`
    *   第一字节：`8B`
    *   指令长度：`2`
    *   保存下列信息为注册码 → 内存方式 → 寄存器 → `EAX`

### 二、选择内存方式

内存地址 → `BF693C` → 点生成，就有你乐的了(这一句是学老师的;)

### 三、内存注册机的使用

由于是针对生成的 `exe` 文件，如果要用注册机，就必须将 `exe` 文件改名为在制作注册机时定义的程序名称（本例制作出的注册机，使用时就应把 `exe` 文件改名为 `crack.exe`），否则不能完成找注册码的工作。找到注册码，只取前面 `16` 字符就可以了！

## 5、我的注册信息

*   受权给：`newlaos`
*   注册键：`EDwKZK84BuzRTYQ2` `4dd466d0e1e0cc123afbe504da2462ca`
