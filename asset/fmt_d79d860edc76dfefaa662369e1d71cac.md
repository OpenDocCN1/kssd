
### 运算流程分析

**运算第一部分**  
`EDX` = `A4D1C46D`  
…… 省略（共 26 次） ……

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

**运算第二部分**  
`EDX` = `51DE003A`  
…… 省略（共 23 次） ……

```
:004E53DE C1EB08           shr ebx, 08  
                           ====>EBX 右移 8 位！  
                           ====>1、EBX=0
```

```
:004E53E1 81E3FFFFFF00     and ebx, 00FFFFFF  
:004E53E7 33D3             xor edx, ebx  
                           ====>1、EDX=A4D1C46D XOR EBX=A4D1C46D  
                           …… 省略（共 26 次） ……

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

                           ====>1、EDX=51DE003A XOR EBX=51DE003A  
                           …… 省略（共 23 次） ……
```

```
:004E53E9 8BDA             mov ebx, edx  
                           ====>EDX 移入 EBX  
                           ====>循环 26 次后，EDX=9E130943  
                           %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  
                           ====>EDX 移入 EBX  
                           ====>循环 23 次后，EDX=E4E8A4FE
```

```
:004E53EB 46               inc esi  
                           ====>ESI 增 1  

:004E53EC 48               dec eax  
                           ====>EAX 减 1  

:004E53ED 75D9             jne 004E53C8  
                           ====>继续循环！共 26 次！  
                           ====>继续循环！共 23 次！
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  `:004E53C1(C)`

```
:004E53EF 8BC3             mov eax, ebx  
                           ====>循环 26 次后，9E130943 入 EAX  
                           %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  
                           ====>循环 23 次后，E4E8A4FE 入 EAX  
```

```
:004E53F1 33D2             xor edx, edx  
:004E53F3 52               push edx  
:004E53F4 50               push eax  
:004E53F5 8D55F4           lea edx, dword ptr [ebp-0C]  
:004E53F8 B808000000       mov eax, 00000008  
:004E53FD E8B63AF2FF       call 00408EB8  
:004E5402 8B45F4           mov eax, dword ptr [ebp-0C]  
:004E5405 8B55F8           mov edx, dword ptr [ebp-08]
:004E5408 E88735F2FF       call 00408994  
                           ====>将 9E130943 中的大写字母转化为小写！  
                           呵呵，9e130943 就是注册码前半部分！  
                           %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  
                           ====>将 E4E8A4FE 中的大写字母转化为小写！  
                           呵呵，e4e8a4fe 就是注册码后半部分！
```

```
:004E540D 33C0             xor eax, eax  
:004E540F 5A               pop edx  
:004E5410 59               pop ecx  
:004E5411 59               pop ecx  
:004E5412 648910           mov dword ptr fs:[eax], edx  
:004E5415 6832544E00       push 004E5432
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  `:004E5430(U)`

```
:004E541A 8D45F4           lea eax, dword ptr [ebp-0C]  
:004E541D E862F0F1FF       call 00404484  
:004E5422 8D45FC           lea eax, dword ptr [ebp-04]  
:004E5425 E85AF0F1FF       call 00404484  
:004E542A C3               ret
```

---

### 真假码比较

1. 进入比较部分！`4E5967` `call 00404890`
2. 进入比较部分！`4E59AE` `call 00404890`

* Referenced by a CALL at Addresses:
  `:0041838F`, `:0041EBD2`, `:004213CF`, `:00427129`, `:00428100`

…… 很多地方 `CALL` 此处！ ……

```
:00404890 53               push ebx  
:00404891 56               push esi  
:00404892 57               push edi  
:00404893 89C6             mov esi, eax  
:00404895 89D7             mov edi, edx  
:00404897 39D0             cmp eax, edx  
                           ====>EAX=1357  
                           ====>EDX=9e130943  
                           %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  
                           ====>EAX=2468  
                           ====>EDX=e4e8a4fe
```

```
:00404899 0F848F000000     je 0040492E  
:0040489F 85F6             test esi, esi  
:004048A1 7468             je 0040490B  
:004048A3 85FF             test edi, edi  
:004048A5 746B             je 00404912  
:004048A7 8B46FC           mov eax, dword ptr [esi-04]  
:004048AA 8B57FC           mov edx, dword ptr [edi-04]  
:004048AD 29D0             sub eax, edx  
:004048AF 7702             ja 004048B3
```

---

### KeyMake 之内存注册机

**目标程序**：`fly{43th}`

**中断点 1**：
- 中断地址：`4E5967`
- 中断次数：1
- 第一字节：`E8`
- 指令长度：5
- 保存下列信息为注册码：
  - 内存方式：`EDX`
  - 结尾插入字符 `-`（`-`前的为注册码的前半部分，`-`后的则为后半部分）

**中断点 2**：
- 中断地址：`4E59AE`
- 中断次数：1
- 第一字节：`E8`
- 指令长度：5
- 内存方式：`EDX`

**修改内存**：
- 修改地址：`4E596C`
- 修改长度：6
- 原始指令：`0F8591020000`
- 修改指令：`909090909090`

---

### 注册信息保存

```registry
REGEDIT4

[HKEY_LOCAL_MACHINE\Software\EZWord]  
"IDcode"="103B2C1E"  
"User"="fly"  
"Code1"="9e130943"  
"Code2"="e4e8a4fe"
```

呵呵，把这些信息存为`.reg`文件导入注册表中应该就是注册版了。^-^

---

### 整理

- 用户名：`fly`
- 申请码：`103B2C1E`
- 注册码：`9e130943` `e4e8a4fe`

---

Cracked By 巢水工作坊——fly【OCN】

2003-2-18 23:34  
`4dd466d0e1e0cc123afbe504da2462ca`
