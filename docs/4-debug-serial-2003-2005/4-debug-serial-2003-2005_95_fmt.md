
# DBReader 1.68

> 原文：[`www.pediy.com/kssd/pediy06/pediy6106.htm`](https://www.pediy.com/kssd/pediy06/pediy6106.htm)

**DBReader 1.68 破解初步分析--我与 MD5 的第一次(失败)**

作者：`newlaos[CCG][DFCG]`

- **软件语言**：简体中文
- **软件类别**：国产软件/共享版/编程工具
- **运行环境**：`Win9x/Me/NT/2000/XP`
- **加入时间**：2003-6-5 7:08:38

**软件介绍**：
`DBReader` 是一个数据库管理工具,通过 `ADO` 能够连接到多种数据库. 通过对数据库的分层次显示,可以浏览,编辑各种数据对象.`DBReader` 可以:
1. 使用 `ADO` 通过 `OLE DB Provider` 或者 `ODBC` 连接到多种数据库.
2. 浏览和编辑各种数据库对象,包括表,视图,字段,索引,以及存储对象定义(现在仅 `sqlserver`).
3. 显示表或视图相关的主键,外键,索引.显示字段的类型,长度,可否为 `null`,等等.
4. 树状显示数据库的结构(表,视图,字段,索引,存储过程,等等).
5. 输入 `SQL` 命令查询数据库,产生可编辑的数据集.
6. 查看 `BLOB` 型字段(`plain text`,`RTF`,`image`).
7. 运行不限长度的 `sql` 脚本,可以从文件载入 `sql` 脚本,也可将 `sql` 脚本存入文件.
8. 调用系统的 `ODBC` 管理功能,配置 `ODBC DSN(Data Source Name)`.

- **加密方式**：注册码
- **功能限制**：次数限制
- **PJ 工具**：`TRW20001.23` 注册版、`W32Dasm8.93` 黄金版，`FI2.5`，`AspackDie 1.41` 汉化版
- **PJ 日期**：2003-06-06

**作者 `newlaos` 申明**：只是学习，请不用于商业用途或是将本文方法制作的注册机任意传播，造成后果，本人一概不负。

1. 先用 `FI2.5` 看一下主文件“`DBReader.exe`”，加了 `ASPACK2.12` 壳，用 `AspackDie 1.41` 汉化版轻松搞定，当然手动也不难，生成脱壳文件 `UNPACKED.EXE`

2. 用 `W32Dasm8.93` 黄金版对 `UNPACKED.EXE` 进行静态反汇编，再用串式数据参考，找到 `"Invalid user/serial"`，双击来到下面代码段。

3. 再用 `TRW20001.23` 注册版进行动态跟踪，下断 `BPX 004D1C9E`(通常在注册成功与否的前面一些下断，这样，才能找到关键部分)，

先输入注册名：`newlaos`

假码: `78787878`

```asm
:004D1C9E 8B1560125300              mov edx, dword ptr [00531260]
:004D1CA4 8B12                      mov edx, dword ptr [edx]
:004D1CA6 E8992AF3FF                call 00404744
:004D1CAB 751A                      jne 004D1CC7  <===这里要跳走
:004D1CAD 6A00                      push 00000000
:004D1CAF 668B0DE41D4D00            mov cx, word ptr [004D1DE4]
:004D1CB6 B202                      mov dl, 02
```

* Possible StringData Ref from Code Obj -> `"Input your username"`
  |
```asm
:004D1CB8 B8F01D4D00                mov eax, 004D1DF0 <===没输入注册名
:004D1CBD E8DA5FF6FF                call 00437C9C
:004D1CC2 E9DE000000                jmp 004D1DA5
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:004D1CAB(C)
  |
```asm
:004D1CC7 8D55F0                    lea edx, dword ptr [ebp-10]
:004D1CCA 8B8310030000              mov eax, dword ptr [ebx+00000310]
:004D1CD0 E83B03FAFF                call 00472010
:004D1CD5 8B45F0                    mov eax, dword ptr [ebp-10]
:004D1CD8 8D55F4                    lea edx, dword ptr [ebp-0C]
:004D1CDB E82C6FF3FF                call 00408C0C
:004D1CE0 8B45F4                    mov eax, dword ptr [ebp-0C]
:004D1CE3 8B1560125300              mov edx, dword ptr [00531260]
:004D1CE9 8B12                      mov edx, dword ptr [edx]
:004D1CEB E8542AF3FF                call 00404744
:004D1CF0 751A                      jne 004D1D0C   <===这里要跳走
:004D1CF2 6A00                      push 00000000
:004D1CF4 668B0DE41D4D00            mov cx, word ptr [004D1DE4]
:004D1CFB B202                      mov dl, 02
```

* Possible StringData Ref from Code Obj -> `"Input your serial"`
  |
```asm
:004D1CFD B80C1E4D00                mov eax, 004D1E0C <===没输入序列号
:004D1D02 E8955FF6FF                call 00437C9C
:004D1D07 E999000000                jmp 004D1DA5
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:004D1CF0(C)
  |
```asm
:004D1D0C 8D55EC                    lea edx, dword ptr [ebp-14]
:004D1D0F 8B8310030000              mov eax, dword ptr [ebx+00000310]
:004D1D15 E8F602FAFF                call 00472010
:004D1D1A 8B45EC                    mov eax, dword ptr [ebp-14] <===EAX=78787878
:004D1D1D 50                        push eax
:004D1D1E 8D55E8                    lea edx, dword ptr [ebp-18]
:004D1D21 8B830C030000              mov eax, dword ptr [ebx+0000030C]
:004D1D27 E8E402FAFF                call 00472010
:004D1D2C 8B55E8                    mov edx, dword ptr [ebp-18] <===EDX=newlaos
:004D1D2F 8B8318030000              mov eax, dword ptr [ebx+00000318]
:004D1D35 59                        pop ecx
:004D1D36 E859FEFFFF                call 004D1B94 <===这里是关键的 CALL,F8 跟进
:004D1D3B 84C0                      test al, al   <===要想正确注册，则 AL 不能为 0
:004D1D3D 750C                      jne 004D1D4B  <===关键跳转了
```

* Possible StringData Ref from Code Obj -> `"Invalid user/serial"`
  |
```asm
:004D1D3F B8281E4D00                mov eax, 004D1E28  <===无效的注册名或密码
:004D1D44 E84B60F6FF                call 00437D94
:004D1D49 EB5A                      jmp 004D1DA5
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:004D1D3D(C)
  |
```asm
:004D1D4B 8D55E4                    lea edx, dword ptr [ebp-1C]
:004D1D4E 8B830C030000              mov eax, dword ptr [ebx+0000030C]
:004D1D54 E8B702FAFF                call 00472010
:004D1D59 8B4DE4                    mov ecx, dword ptr [ebp-1C]
```

* Possible StringData Ref from Code Obj -> `"user"`
  |
```asm
:004D1D5C BA441E4D00                mov edx, 004D1E44
:004D1D61 8B8318030000              mov eax, dword ptr [ebx+00000318]
:004D1D67 E800FBFFFF                call 004D186C
:004D1D6C 8D55E0                    lea edx, dword ptr [ebp-20]
:004D1D6F 8B8310030000              mov eax, dword ptr [ebx+00000310]
:004D1D75 E89602FAFF                call 00472010
:004D1D7A 8B4DE0                    mov ecx, dword ptr [ebp-20]
```

* Possible StringData Ref from Code Obj -> `"serial"`
  |
```asm
:004D1D7D BA541E4D00                mov edx, 004D1E54
:004D1D82 8B8318030000              mov eax, dword ptr [ebx+00000318]
:004D1D88 E8DFFAFFFF                call 004D186C
```

* Possible StringData Ref from Code Obj -> `"Register success"`
  |
```asm
:004D1D8D B8641E4D00                mov eax, 004D1E64 <===注册成功
:004D1D92 E8FD5FF6FF                call 00437D94
:004D1D97 C6831C03000001            mov byte ptr [ebx+0000031C], 01
:004D1D9E 8BC3                      mov eax, ebx
:004D1DA0 E80FCCFBFF                call 0048E9B4
```

-----------------:004D1D36 call 004D1B94 关键的 CALL,F8 跟进------------------

```asm
:004D1B94 55                          push ebp
:004D1B95 8BEC                        mov ebp, esp
:004D1B97 6A00                        push 00000000
:004D1B99 6A00                        push 00000000
:004D1B9B 6A00                        push 00000000
:004D1B9D 6A00                        push 00000000
:004D1B9F 6A00                        push 00000000
:004D1BA1 53                          push ebx
:004D1BA2 56                          push esi
:004D1BA3 894DF8                      mov dword ptr [ebp-08], ecx
:004D1BA6 8955FC                      mov dword ptr [ebp-04], edx
:004D1BA9 8BF0                        mov esi, eax
:004D1BAB 8B45FC                      mov eax, dword ptr [ebp-04]
:004D1BAE E8352CF3FF                  call 004047E8
:004D1BB3 8B45F8                      mov eax, dword ptr [ebp-08]
:004D1BB6 E82D2CF3FF                  call 004047E8
:004D1BBB 33C0                        xor eax, eax
:004D1BBD 55                          push ebp
:004D1BBE 68241C4D00                  push 004D1C24
:004D1BC3 64FF30                      push dword ptr fs:[eax]
:004D1BC6 648920                      mov dword ptr fs:[eax], esp
:004D1BC9 33DB                        xor ebx, ebx
:004D1BCB 8D55F4                      lea edx, dword ptr [ebp-0C]
:004D1BCE 8B45FC                      mov eax, dword ptr [ebp-04] <===EAX=newlaos
:004D1BD1 E83670F3FF                  call 00408C0C  <===算出注册名的长度为 7
:004D1BD6 837DF400                    cmp dword ptr [ebp-0C], 00000000
:004D1BDA 742D                       je 004D1C09 <===如果注册名没输入，这里就跳向 OVER 了
:004D1BDC 8D55F0                      lea edx, dword ptr [ebp-10]
:004D1BDF 8B45F8                      mov eax, dword ptr [ebp-08] <===EAX=78787878
:004D1BE2 E82570F3FF                  call 00408C0C  <===算出注册码的长度
:004D1BE7 837DF000                    cmp dword ptr [ebp-10], 00000000
:004D1BEB 741C                       je 004D1C09 <===如果注册码没输入，这里就跳向 OVER 了
```



`lea ecx, dword ptr [ebp\-14]`  

`mov edx, dword ptr [ebp\-04]` <===`EDX`=newlaos  

`mov eax, esi`  

`call 004D1AC8` <===这个就算法 `CALL` 了，`F8` 跟进  

`mov edx, dword ptr [ebp\-14]` <===`EDX` 就出来了注册码 `b577a94d731` 了  

`mov eax, dword ptr [ebp\-08]` <===`EAX`=78787878 我们输入的假码  

`call 00404744` <===拿真码和假作对比  

`jne 004D1C09` <===如果不对就跳向 `OVER` 了  

`mov bl, 01`  

.......  

`----:004D1BF5 call 004D1AC8 算法 CALL 了，F8 跟进------------------`  

`push ebp`  

`mov ebp, esp`  

`add esp, FFFFFFE0`  

`push ebx`  

`push esi`  

`push edi`  

`xor ebx, ebx`  

`mov dword ptr [ebp\-20], ebx`  

`mov dword ptr [ebp\-08], ebx`  

`mov dword ptr [ebp\-0C], ebx`  

`mov edi, ecx`  

`mov dword ptr [ebp\-04], edx`  

`mov eax, dword ptr [ebp\-04]`  

`call 004047E8`  

`xor eax, eax`  

`push ebp`  

`push 004D1B84`  

`push dword ptr fs:[eax]`  

`mov dword ptr fs:[eax], esp`  

`lea eax, dword ptr [ebp\-0C]`  

`call 00404338`  

`lea edx, dword ptr [ebp\-1C]`  

`mov eax, dword ptr [ebp\-04]` <===`EAX`=newlaos  

`call 004D1050` <===这个就是关键的 `CALL` 了，`F8` 跟进  

`lea eax, dword ptr [ebp\-1C]`  

`lea edx, dword ptr [ebp\-08]`  

`call 004D10C4` <===这个 `CALL`，只是将上面的 `CALL` 的数值转为注册码的初形(也就是 MD5 码的值了)  

`mov eax, dword ptr [ebp\-08]` <===`EAX`=bc581765755a299334dfd8376d3be1a9  

`call 004045F8`  

`mov esi, eax`  

`dec esi`  

`test esi, esi`  

`jl 004D1B57`  

`inc esi`  

`xor ebx, ebx`  

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
`:004D1B55(C)`  

`mov eax, ebx`  

`mov ecx, 00000003`  

`cdq`  

`idiv ecx`  

`test edx, edx`  

`jne 004D1B53`  

`lea eax, dword ptr [ebp\-20]`  

`push eax`  

`mov ecx, 00000001`  

`mov edx, ebx`  

`mov eax, dword ptr [ebp\-08]` <===`EAX`=bc581765755a299334dfd8376d3be1a9  

`call 00404858`  

`mov edx, dword ptr [ebp\-20]`  

`lea eax, dword ptr [ebp\-0C]`  

`call 00404600`  

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
`:004D1B33(C)`  

`inc ebx`  

`dec esi`  

`jne 004D1B27` <===向上跳构成循环结构,只从第一个要取，之后，就是 3 个整除位取值构成最后注册码。也就是 `b577a94d731`  

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
`:004D1B22(C)`  

`mov eax, edi`  

`mov edx, dword ptr [ebp\-0C]`  

.......  

`----:004D1B05 call 004D1050 关键的 CALL 了，F8 跟进-------`  

`push ebp`  

`mov ebp, esp`  

`add esp, FFFFFFA4`  

`push ebx`  

`mov ebx, edx`  

`mov dword ptr [ebp\-04], eax`  

`mov eax, dword ptr [ebp\-04]`  

`call 004047E8`  

`xor eax, eax`  

`push ebp`  

`push 004D10B6`  

`push dword ptr fs:[eax]`  

`mov dword ptr fs:[eax], esp`  

`lea eax, dword ptr [ebp\-5C]`  

`call 004D0F28`  

`mov eax, dword ptr [ebp\-04]` <===`EAX`=newlaos  

`call 004045F8` <===算出长度 `EAX`=7  

`push eax`  

`mov eax, dword ptr [ebp\-04]`  

`call 004047F8`  

`mov edx, eax`  

`lea eax, dword ptr [ebp\-5C]`  

<===这里的值是(标准的 MD5 加密算法的初始数值)，但似乎作者又做也修改？？？  

`A`=0x01234567 --->`A`=0x12345678  

`B`=0x89abcdef --->`B`=0x89abcdef  

`C`=0xfedcba98 --->`C`=0xfedcba98   

`D`=0x76543210 --->`D`=0x87654321 

`pop ecx` <===为注册名的长度 `ECX`=7  

`call 004D0F5C` <===将注册码的 ASC 值组成一个大数字 `6E65776C616F73`  

`mov edx, ebx`  

`lea eax, dword ptr [ebp\-5C]`  

`call 004D0FDC` <===算法 `CALL` 了，`F8` 再跟进(晕，竟然是 MD5 的加密算法)  

`xor eax, eax`  

`pop edx`  

`pop ecx`  

`pop ecx`  

`mov dword ptr fs:[eax], edx`  

`push 004D10BD`  

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
`:004D10BB(U)`  

`lea eax, dword ptr [ebp\-04]`  

`call 00404338`  

`ret`  

.......  

`----:004D109B call 004D0FDC 算法 CALL 了，F8 再跟进--------`  

`push ebx`  

`push esi`  

`add esp, FFFFFFF8`  

`mov esi, edx`  

`mov ebx, eax`  

`mov edx, esp`  

`lea eax, dword ptr [ebx+10]`  

`mov ecx, 00000002`  

`call 004D07BC` <===是这里算出来的 `0x38`  

`mov eax, dword ptr [ebx+10]` <===`EAX`=0x38(程序内定？)  

`shr eax, 03`  

`and eax, 0000003F`  

`cmp eax, 00000038`  

`jnb 004D100D`  

`mov edx, 00000038`  

`sub edx, eax`  

`mov eax, edx`  

`jmp 004D1016` <===这里跳走  

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
`:004D1000(C)`  

`mov edx, 00000078`  



```
:004D1012 2BD0 sub edx, eax
:004D1014 8BC2 mov eax, edx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004D100B(U)

* Possible StringData Ref from Data Obj -> "€"

:004D1016 BA74FA5200 mov edx, 0052FA74
:004D101B 8BCB mov ecx, ebx
:004D101D 91 xchg eax, ecx
:004D101E E839FFFFFF call 004D0F5C
:004D1023 8BD4 mov edx, esp
:004D1025 8BC3 mov eax, ebx
:004D1027 B908000000 mov ecx, 00000008
:004D102C E82BFFFFFF call 004D0F5C <===到这里就算出来了,F8 再跟进
:004D1031 8BD6 mov edx, esi
:004D1033 8BC3 mov eax, ebx
:004D1035 B904000000 mov ecx, 00000004
:004D103A E87DF7FFFF call 004D07BC
:004D103F 8BC3 mov eax, ebx
:004D1041 BA58000000 mov edx, 00000058
:004D1046 E80D66F3FF call 00407658
:004D104B 59 pop ecx
:004D104C 5A pop edx
:004D104D 5E pop esi
:004D104E 5B pop ebx
:004D104F C3 ret
```

...

...

```
:004D102C call 004D0F5C 到这里就算出来了,F8 再跟进
:004D0F5C 53 push ebx
:004D0F5D 56 push esi
:004D0F5E 57 push edi
:004D0F5F 55 push ebp
:004D0F60 8BF9 mov edi, ecx
:004D0F62 8BEA mov ebp, edx
:004D0F64 8BF0 mov esi, eax
:004D0F66 8B4610 mov eax, dword ptr [esi+10]
:004D0F69 C1E803 shr eax, 03
:004D0F6C 83E03F and eax, 0000003F
:004D0F6F 8BD7 mov edx, edi
:004D0F71 C1E203 shl edx, 03
:004D0F74 015610 add dword ptr [esi+10], edx
:004D0F77 3B5610 cmp edx, dword ptr [esi+10]
:004D0F7A 7603 jbe 004D0F7F
:004D0F7C FF4614 inc [esi+14]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004D0F7A(C)

:004D0F7F 8BD7 mov edx, edi
:004D0F81 C1EA1D shr edx, 1D
:004D0F84 015614 add dword ptr [esi+14], edx
:004D0F87 BB40000000 mov ebx, 00000040
:004D0F8C 2BD8 sub ebx, eax
:004D0F8E 3BDF cmp ebx, edi
:004D0F90 7732 ja 004D0FC4
:004D0F92 8D440618 lea eax, dword ptr [esi+eax+18]
:004D0F96 8BCB mov ecx, ebx
:004D0F98 8BD5 mov edx, ebp
:004D0F9A E8B166F3FF call 00407650
:004D0F9F 8BD6 mov edx, esi <===EDX 就是 MD5 的初始值
:004D0FA1 8D4618 lea eax, dword ptr [esi+18] <===EAX 指向的就是 6E65776C616F7380(注：后面加了个 80)
:004D0FA4 E84FF8FFFF call 004D07F8 <===呵呵，就是它了，问题是怎么会是这样呢？只有 F8 再跟进了
:004D0FA9 EB0E jmp 004D0FB9

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004D0FBE(C)

:......
```

...

...

```
:004D0FA4 call 004D07F8 只有 F8 再跟进了
```

### 4、用 `KEYMAKE1.73` 制作内存注册机

- 下断：`004D1BFD`
- 断次数：1
- 指令长度：3
- 指令首字：`8B`
- 保存内存：`EDX` 就可以了

### 5、说明

很随便地拿来破的一个软件，开始看到是明码比较，以为可以很快搞定，谁知让我第一次失败在 `MD5` 加密里(以前是没有遇到过)，惨呀!毕竟对 `MD5` 的实现不明白，只能做出个内存注册机来。----失败!

哪位兄弟有兴趣也看看吧

`4dd466d0e1e0cc123afbe504da2462ca`
