
# smadinst

> 原文：[`www.pediy.com/kssd/pediy03/forum499.htm`](https://www.pediy.com/kssd/pediy03/forum499.htm)

我破解了一个小游戏（`http://www.inhua.com/down/smadinst.exe`），但没有完全破解它，想请高手指点一下！！！

我先用 `UNASPACK` 脱壳，用 `W32dasm` 反汇编和 `TRW2000` 调试后，我共修改了三处：

1. `:00473C30` `7432` je `00473C64`
   改 `EB32`
2. `:00473D52` `756A` jne `00473DBE`
   改 `9090`
3. `:00473D5A` `7E62` jle `00473DBE`
   改 `9090`

以上的修改，只是使当时的注册显示成功，以后再开始仍是未注册版本，还要重新注册才行。这是为何？想请高手指点一下！！！

我输入的信息是:

`name`: `luoyi`

`number`: `654321` 化成 `16` 进制就是下面总是出现的 `9fbf1` 了!

注: 以下所有数字, 未经特别说明, 都为 `16` 进制!

```
015F:00473D0D  MOV     EAX,ESI          eax=9fbf1
015F:00473D0F  MOV     ECX,2710        ecx=2710
015F:00473D14  CDQ
015F:00473D15  IDIV    ECX            eax/ecx     商----〉eax，余数 ----〉edx
015F:00473D17  MOV     EDI,EDX       余数----〉edi
015F:00473D19  MOV     ECX,EDI
015F:00473D1B  XOR     ECX,0C01
015F:00473D21  MOV     EDX,00473ED4
015F:00473D26  MOV     EAX,[004919E0]
015F:00473D2B  CALL    0045E560
015F:00473D30  MOV     EAX,[004919E0]
015F:00473D35  CALL    00402E2C
015F:00473D3A  MOV     EAX,EDI        eax=第一次除法的余数
015F:00473D3C  MOV     ECX,33        ecx=33
015F:00473D41  CDQ
015F:00473D42  IDIV    ECX           eax/ecx
015F:00473D44  MOV     ECX,EAX       第二次的商----〉ecx
015F:00473D46  MOV     EAX,ESI       eax=9fbf1
015F:00473D48  MOV     EDI,2710       edi=2710
015F:00473D4D  CDQ
015F:00473D4E  IDIV    EDI           eax/edi  eax 中是第三次的商
015F:00473D50  CMP     EAX,ECX       如果第二次的商等于第三次的商，就成功了！
015F:00473D52  JNZ     00473DBE
```

算法整理：

假设你输入的是 `A`，则:

`B` = `A` / `2710` 的余数.  
`C` = `A` / `2710` 的商  

`D` = `B` / `33` 的商  

`C` = `D` 时注册码为有效.

先对主程序脱壳，然后反编译，得到字符串“`Thank you`”地址为 `00473D7A`，字符串“`This is not a valid`”地址为 `00473C32`、`00473DBE`。相关代码如下：

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`|:00473C30(C)`

```
:00473C64 8D55F8                 lea edx, dword ptr [ebp-08]
:00473C67 8B8338020000           mov eax, dword ptr [ebx+00000238]
:00473C6D E8A2E5FAFF             call 00422214
:00473C72 8B55F8                 mov edx, dword ptr [ebp-08]
:00473C75 8D85F8FEFFFF           lea eax, dword ptr [ebp+FFFFFEF8]
:00473C7B B9FF000000             mov ecx, 000000FF
:00473C80 E8ABFFF8FF             call 00403C30
:00473C85 8D95F8FEFFFF           lea edx, dword ptr [ebp+FFFFFEF8]
:00473C8B B8E4194900             mov eax, 004919E4
:00473C90 B128                   mov cl, 28
:00473C92 E8F5ECF8FF             call 0040298C
:00473C97 B201                   mov dl, 01
:00473C99 A1F4E04500             mov eax, dword ptr [0045E0F4]
:00473C9E E885A5FEFF             call 0045E228
:00473CA3 A3E0194900             mov dword ptr [004919E0], eax
:00473CA8 B101                   mov cl, 01
```

* Possible StringData Ref from Code Obj ->"`\Software\StarMadness`"

```
:00473CAA BA683E4700             mov edx, 00473E68
:00473CAF A1E0194900             mov eax, dword ptr [004919E0]
:00473CB4 E867A6FEFF             call 0045E320
:00473CB9 8D55F8                 lea edx, dword ptr [ebp-08]
:00473CBC 8B8338020000           mov eax, dword ptr [ebx+00000238]
:00473CC2 E84DE5FAFF             call 00422214
:00473CC7 8B4DF8                 mov ecx, dword ptr [ebp-08]
```

* Possible StringData Ref from Code Obj ->"`UserName`"

```
:00473CCA BA883E4700             mov edx, 00473E88
:00473CCF A1E0194900             mov eax, dword ptr [004919E0]
:00473CD4 E8E3A7FEFF             call 0045E4BC
:00473CD9 8D55F8                 lea edx, dword ptr [ebp-08]
:00473CDC 8B833C020000           mov eax, dword ptr [ebx+0000023C]
:00473CE2 E82DE5FAFF             call 00422214
:00473CE7 8B4DF8                 mov ecx, dword ptr [ebp-08]
```

* Possible StringData Ref from Code Obj ->"`SerialNumber`"

```
:00473CEA BA9C3E4700             mov edx, 00473E9C
:00473CEF A1E0194900             mov eax, dword ptr [004919E0]
:00473CF4 E8C3A7FEFF             call 0045E4BC
```

* Possible StringData Ref from Code Obj ->"`1.28`"

* Possible StringData Ref from Code Obj ->"`Version`"

```
:00473CF9 B9B43E4700             mov ecx, 00473EB4
:00473CFE BAC43E4700             mov edx, 00473EC4
:00473D03 A1E0194900             mov eax, dword ptr [004919E0]
:00473D08 E8AFA7FEFF             call 0045E4BC
:00473D0D 8BC6                   mov eax, esi
:00473D0F B910270000             mov ecx, 00002710
:00473D14 99                     cdq
:00473D15 F7F9                   idiv ecx
:00473D17 8BFA                   mov edi, edx
:00473D19 8BCF                   mov ecx, edi
:00473D1B 81F1010C0000           xor ecx, 00000C01
```

* Possible StringData Ref from Code Obj ->"`BuildID`"

```
:00473D21 BAD43E4700             mov edx, 00473ED4
:00473D26 A1E0194900             mov eax, dword ptr [004919E0]
:00473D2B E830A8FEFF             call 0045E560
:00473D30 A1E0194900             mov eax, dword ptr [004919E0]
:00473D35 E8F2F0F8FF             call 00402E2C
:00473D3A 8BC7                   mov eax, edi
:00473D3C B933000000             mov ecx, 00000033
:00473D41 99                     cdq
:00473D42 F7F9                   idiv ecx
:00473D44 8BC8                   mov ecx, eax
:00473D46 8BC6                   mov eax, esi
:00473D48 BF10270000             mov edi, 00002710
:00473D4D 99                     cdq
:00473D4E F7FF                   idiv edi
:00473D50 3BC1                   cmp eax, ecx    ; 比较，若不正确，跳到“This is not a valid”处
:00473D52 756A                   jne 00473DBE
:00473D54 81FE10270000           cmp esi, 00002710  ; 同上
:00473D5A 7E62                   jle 00473DBE
:00473D5C C605D8C2470001          mov byte ptr [0047C2D8], 01
:00473D63 C605DC19490001          mov byte ptr [004919DC], 01
```

* Possible StringData Ref from Code Obj ->"`JOIN GAME`"

```
:00473D6A BAE43E4700             mov edx, 00473EE4
:00473D6F 8B835C030000           mov eax, dword ptr [ebx+0000035C]
:00473D75 E8DAE4FAFF             call 00422254
```

* Possible StringData Ref from Code Obj ->"`Thank you,`"

```
:00473D7A BAF83E4700             mov edx, 00473EF8
:00473D7F 8B8374030000           mov eax, dword ptr [ebx+00000374]
:00473D85 E8CAE4FAFF             call 00422254
```

* Possible StringData Ref from Code Obj ->"`have a nice game!`"

```
:00473D8A BA0C3F4700             mov edx, 00473F0C
:00473D8F 8B837C030000           mov eax, dword ptr [ebx+0000037C]
:00473D95 E8BAE4FAFF             call 00422254
:00473D9A B201                   mov dl, 01
:00473D9C 8B8370030000           mov eax, dword ptr [ebx+00000370]
:00473DA2 E835E3FAFF             call 004220DC
:00473DA7 C605FCC2470001          mov byte ptr [0047C2FC], 01
:00473DAE C60500C347000C          mov byte ptr [0047C300], 0C
:00473DB5 C605D8FE480014          mov byte ptr [0048FED8], 14
:00473DBC EB2F                   jmp 00473DED
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`|:00473D52(C), :00473D5A(C)`

* Possible StringData Ref from Code Obj ->"`This is not a valid`"

```
:00473DBE BA343E4700             mov edx, 00473E34
:00473DC3 8B8374030000           mov eax, dword ptr [ebx+00000374]
```



`00473DC9` `E886E4FAFF` 调用 `00422254`

* 可能的字符串数据引用来自代码对象 -> `"serial number!"`

`00473DCE` `BA503E4700` `mov edx, 00473E50`

`00473DD3` `8B837C030000` `mov eax, dword ptr [ebx+0000037C]`

`00473DD9` `E876E4FAFF` 调用 `00422254`

`00473DDE` `B201` `mov dl, 01`

`00473DE0` `8B8370030000` `mov eax, dword ptr [ebx+00000370]`

`00473DE6` `E8F1E2FAFF` 调用 `004220DC`

`00473DEB` `EB1A` `jmp 00473E07`

找到这段代码，就好分析了。

`0046F6A7` `0FB638` `movzx edi, byte ptr [eax]` ; `eax` -> 用户名

`0046F6AA` `0FAFFE` `imul edi, esi`

`0046F6AD` `03CF` `add ecx, edi`

`0046F6AF` `46` `inc esi`

`0046F6B0` `40` `inc eax`

`0046F6B1` `4A` `dec edx`

`0046F6B2` `75F3` `jne 0046F6A7`

; 上面这段代码将用户名进行累加：第 1 字符 * 1 + 第 2 字符 * 2 + 第 3 字符 * 3 + ...... 并存入 `ECX` 中。

`0046F6B4` `8BC1` `mov eax, ecx`

`0046F6B6` `B9E8030000` `mov ecx, 000003E8`

`0046F6BB` `99` `cdq`

`0046F6BC` `F7F9` `idiv ecx`

`0046F6BE` `8955C8` `mov dword ptr [ebp-38], edx`

`0046F6C1` `DB45C8` `fild dword ptr [ebp-38]`

`0046F6C4` `DB2D44FF4600` `fld tbyte ptr [0046FF44]`

`0046F6CA` `DEC9` `fmulp st(1), st(0)`

`0046F6CC` `D80550FF4600` `fadd dword ptr [0046FF50]`

`0046F6D2` `DD5DF0` `fstp qword ptr [ebp-10]`

`0046F6D5` `9B` `wait`

`0046F6D6` `A13C1A4900` `mov eax, dword ptr [00491A3C]`

`0046F6DB` `35010C0000` `xor eax, 00000C01`

`0046F6E0` `8945C8` `mov dword ptr [ebp-38], eax`

`0046F6E3` `DB45C8` `fild dword ptr [ebp-38]`

`0046F6E6` `DB2D54FF4600` `fld tbyte ptr [0046FF54]`

`0046F6EC` `DEC9` `fmulp st(1), st(0)`

`0046F6EE` `DD5DE8` `fstp qword ptr [ebp-18]`

`0046F6F1` `9B` `wait`

`0046F6F2` `DD45E8` `fld qword ptr [ebp-18]`

`0046F6F5` `E8CE31F9FF` 调用 `004028C8`

`0046F6FA` `DD45F0` `fld qword ptr [ebp-10]`

`0046F6FD` `DC4DE8` `fmul qword ptr [ebp-18]`

`0046F700` `DEE9` `fsubp st(1), st(0)`

`0046F702` `DD5DF8` `fstp qword ptr [ebp-08]`

`0046F705` `9B` `wait`

`0046F706` `DD45F8` `fld qword ptr [ebp-08]`

`0046F709` `D9E1` `fabs`

`0046F70B` `DB2D44FF4600` `fld tbyte ptr [0046FF44]`

`0046F711` `DED9` `fcompp`

`0046F713` `DFE0` `fstsw ax`

`0046F715` `9E` `sahf`

`0046F716` `7607` `jbe 0046F71F` ; 这里就是关键的检测点，不跳即可成功注册！

`0046F718` `C605D8C2470001` `mov byte ptr [0047C2D8], 01` ; 即将内存 `[0047C2D8]` 处置为 1

; 上面这段代码有许多浮点运行指令，看不懂。哪位看得懂的，给解释一下。

`0046F71F` `B201` `mov dl, 01`

`0046F721` `A1342C4400` `mov eax, dword ptr [00442C34]`

`0046F726` `E8813EFDFF` 调用 `004435AC`

`0046F72B` `A3580B4900` `mov dword ptr [00490B58], eax`

`0046F730` `6A08` `push 00000008`

`0046F732` `B9E0010000` `mov ecx, 000001E0`

`0046F737` `BA80020000` `mov edx, 00000280`

`0046F73C` `A1580B4900` `mov eax, dword ptr [00490B58]`

`0046F741` `E8D64DFDFF` 调用 `0044451C`

`0046F746` `B201` `mov dl, 01`

说明：`0046F716` `jbe 0046F71F` 处即为关键检测点，不跳即可成功注册。

`0046F6A7` `0FB638` `movzx edi, byte ptr [eax]` ; `eax` -> 用户名

`0046F6AA` `0FAFFE` `imul edi, esi`

`0046F6AD` `03CF` `add ecx, edi`

`0046F6AF` `46` `inc esi`

`0046F6B0` `40` `inc eax`

`0046F6B1` `4A` `dec edx`

`0046F6B2` `75F3` `jne 0046F6A7`

; 上面这段代码将用户名进行累加：第 1 字符 * 1 + 第 2 字符 * 2 + 第 3 字符 * 3 + ...... 并存入 `ECX` 中。

`0046F6B4` `8BC1` `mov eax, ecx`

`0046F6B6` `B9E8030000` `mov ecx, 000003E8` ====> 除以 1000

`0046F6BB` `99` `cdq`

`0046F6BC` `F7F9` `idiv ecx`

`0046F6BE` `8955C8` `mov dword ptr [ebp-38], edx` ====> 余数存放此处

此时在 `SoftICE` 下用 `WF` 命令打开浮点寄存器

`0046F6C1` `DB45C8` `fild dword ptr [ebp-38]` ====> `ST0`=余数值

`0046F6C4` `DB2D44FF4600` `fld tbyte ptr [0046FF44]` ====> `ST0`=1E-3 `ST1`=余数值

`0046F6CA` `DEC9` `fmulp st(1), st(0)` ====> 将余数乘以 1E-3

`0046F6CC` `D80550FF4600` `fadd dword ptr [0046FF50]` ====> 结果 + 1

`0046F6D2` `DD5DF0` `fstp qword ptr [ebp-10]` ====> 存放到 `[ebp-10]` 处

`0046F6D5` `9B` `wait`

`0046F6D6` `A13C1A4900` `mov eax, dword ptr [00491A3C]` ====> `eax`=BuildID

`0046F6DB` `35010C0000` `xor eax, 00000C01` ====> `eax`=序列号除以 10000 的余数

`0046F6E0` `8945C8` `mov dword ptr [ebp-38], eax` ====> 存于此处

`0046F6E3` `DB45C8` `fild dword ptr [ebp-38]` ====> `ST0`=余数值

`0046F6E6` `DB2D54FF4600` `fld tbyte ptr [0046FF54]` ====> `ST0`=1E-4 `ST1`=余数值

`0046F6EC` `DEC9` `fmulp st(1), st(0)` ====> 余数乘以 1E-4

`0046F6EE` `DD5DE8` `fstp qword ptr [ebp-18]` ====> 结果保存在 `[ebp-18]`

`0046F6F1` `9B` `wait`

`0046F6F2` `DD45E8` `fld qword ptr [ebp-18]` ====> `ST0`=`[ebp-18]`，即注册码的余数除以 10000(或乘以 1E-4)

`0046F6F5` `E8CE31F9FF` 调用 `004028C8` ====> 余数的 `COS` 值

`0046F6FA` `DD45F0` `fld qword ptr [ebp-10]` ====> `ST0`=由用户名生成的余数的计算值

`0046F6FD` `DC4DE8` `fmul qword ptr [ebp-18]` ====> 乘以 `[ebp-18]`

`0046F700` `DEE9` `fsubp st(1), st(0)` ====> 余数的 `COS` 值 - 刚才的乘积值

`0046F702` `DD5DF8` `fstp qword ptr [ebp-08]` ====> 结果保存在 `[ebp-8]`

`0046F705` `9B` `wait`

`0046F706` `DD45F8` `fld qword ptr [ebp-08]`

`0046F709` `D9E1` `fabs`

`0046F70B` `DB2D44FF4600` `fld tbyte ptr [0046FF44]` ====> `ST0`=1E-3，`ST1`=`[ebp-8]`

`0046F711` `DED9` `fcompp` ====> `ST0` 与 `ST1` 比较

`0046F713` `DFE0` `fstsw ax`

`0046F715` `9E` `sahf`

`0046F716` `7607` `jbe 0046F71F` ====> 小于等于则转

; 这里就是关键的检测点，不跳即可成功注册！

`0046F718` `C605D8C2470001` `mov byte ptr [0047C2D8], 01` ; 即将内存 `[0047C2D8]` 处置为 1

; 上面这段代码有许多浮点运行指令，看不懂。哪位看得懂的，给解释一下。

`0046F71F` `B201` `mov dl, 01`

`0046F721` `A1342C4400` `mov eax, dword ptr [00442C34]`

`0046F726` `E8813EFDFF` 调用 `004435AC`

`0046F72B` `A3580B4900` `mov dword ptr [00490B58], eax`

`0046F730` `6A08` `push 00000008`

`0046F732` `B9E0010000` `mov ecx, 000001E0`

`0046F737` `BA80020000` `mov edx, 00000280`

`0046F73C` `A1580B4900` `mov eax, dword ptr [00490B58]`

`0046F741` `E8D64DFDFF` 调用 `0044451C`

`0046F746` `B201` `mov dl, 01`

说明：`0046F716` `jbe 0046F71F` 处即为关键检测点，不跳即可成功注册。

从上面可以看出，比较的过程如下：

* 将用户名进行累加：第 1 字符 * 1 + 第 2 字符 * 2 + 第 3 字符 * 3 + ...，结果除以 1000，取余数，再除以 1000，再加上 1
* 序列号 / 10000 的余数再除以 10000 的 `COS` 值 - 序列号的余数 / 10000 * 由用户名生成的余数的计算值的绝对值小于 `1E-3`

下面开始反推：

如果用户名为 `robot`，则执行到 `46F6B4` 时 `ecx=0x676`，余数为 `28E` (即 654)，`654/1000 + 1 = 1.654`

设序列号的余数值为 `x`，则 `x * 1.654 = cos(x)`，则 `x = 0.5236`，`0.5236 * 10000 = 5236`，`5236 mod 51 = 102`

因此，注册码为：`1025236`

重新整理：用户名：`robot` 注册码：`1025236`，注册后运行程序，已经变为注册版了，破解成功

注：上面的方程求解是用 `MathCad` 求出的 `4dd466d0e1e0cc123afbe504da2462ca`
