
这回是取输入的注册码，调用`GetWindowTextA`的地址还是上面得到的那个，也加个分析提示`/*call(77D1216B,4C80FC)*/`，这里可以看出来注册码必须是`0x28`（40）位。

```
00496DC7  |.  32  vWriteMemSs4   "数组长度" EXIT DWORD v212 = 14
00496D91  |.  32  vWriteMemSs4   "数组" EXIT DWORD v213 = 0FFFFFFFC
004D2528  |.  32  vWriteMemSs4   "字符串长度" EXIT DWORD v214 = 28
004938F7  |.  BF  vWriteMemSs4   "字符串" EXIT DWORD v215 = 0FFFFFFAC//输入的注册码
...
0044C1AA  |.  B5  vRet           call v221; <VMPCrack.16进制字符串转数组>//把输入的注册码字符串转成数组
00486E37  |.  94  vReadMemSs4    DWORD m56 = DWORD SS:[Je(AndFlag("结果长度", "结果长度")) + 0FFFFFF50]
00486E04  |.  F1  vJmp_00412430  if ("结果长度" == 0) goto VMPCrack.00441B20//结果长度为0表示转换失败，注册码不是16进制字符串，转到弹出提示"注册码错误。"
```

这里调用了一个函数`4024F0`，是把16进制字符串转成数组，说明注册码必须是16进制字符串，注册码输入`1234567812345678123456781234567812345678`再往下跟。

```
0042E566  |.  B9  vJmp_00411670   callVM <VMPCrack.验证注册码>		//调用验证函数
004BE669  |.  34  vReadMemSs4     DWORD m57 = DWORD SS:[Jnz(AndFlag(GetBytes("验证成功", 0, 1), GetBytes("验证成功", 0, 1))) + 0FFFFFF50]
004BE637  |.  05  vJmp_00412430   if (GetBytes("验证成功", 0, 1) == 0) goto VMPCrack.00441B20//返回0为验证失败，转到弹出提示"注册码错误。"
004B66A0  |.  BF  vWriteMemSs4    Style EXIT DWORD v231 = 0
00454109  |.  32  vWriteMemSs4    Title EXIT DWORD v232 = 4074EC	//标题"VMPCrackMe"
00491BBF  |.  32  vWriteMemSs4    Text EXIT DWORD v233 = 1		//内容，地址为1？
00468648  |.  E4  vWriteMemSs4    hOwner EXIT DWORD v234 = v182
004D80F4  |.  09  vJmp_00411C30   callVM <VMPCrack.MessageBoxA>		//弹出消息框
...
0046DC11  |.  16  vRet            return v246
```

`42E566`的调用像是关键的验证函数，因为他返回0就转到验证失败了，试试能不能在`if (GetBytes("验证成功", 0, 1) == 0)`爆破。可以执行到`vJmp`然后修改堆栈里的目标，或者先执行到验证失败的转移目标，再用菜单里的“此处为新vEIP”功能设置到正确的目标。执行后出错了，`77d08944`引用内存`1`，好像是调用`MessageBoxA`时出的，回去看看参数，内容的地址果然是`1`。那怎么显示消息？先不管他，跟进验证函数里看看。

先试试最简单的方法，看能不能在验证函数里爆破。因为知道了返回0是失败，在函数里找出所有返回，可以查找`vRet`（不包括用作调用的），一共找到6个，看看返回值`EAX`。

```
00475768  |.  ED  vPushReg4 vR0   EAX DWORD v84 = GetBytes(v80, 1, 3) : 0
...
0047575D  |.  2E  vRet            return v83
//
004C8E1B  |.  F5  vPushReg4 vR2   EAX DWORD v95 = GetBytes(s43, 1, 3) : 0
...
004C8E10  |.  2E  vRet            return v94
//
0049DE80  |.  ED  vPushReg4 vR0   EAX DWORD v125 = GetBytes(s46, 1, 3) : 0
...
0049DE75  |.  2E  vRet            return v124
//
004A22AC  |.  05  vPushReg4 vR6   EAX DWORD v140 = GetBytes(entryVMEax_4689BB, 1, 3) : 1
...
004A22A1  |.  2E  vRet            return v139
//
0048704D  |.  11  vPushReg4 vR9   EAX DWORD v151 = GetBytes(entryVMEax_45CED7, 1, 3) : 0
...
00487042  |.  37  vRet            return v150
//
004376C3  |.  FD  vPushReg4 vR4   EAX DWORD v162 = GetBytes(v120, 1, 3) : 0
...
004376B8  |.  2E  vRet            return v161
```

其中5个把`EAX`的低`BYTE`（`AL`）设为0，另一个设为1，应该就是验证成功的地方。找一找转移到这些返回的分支，可以在包含返回的指令块开始用右键菜单“转到-转移来自”，看看是哪里跳过来的，最后找到下面这些。

```
004425EA  |. /06  vJmp_00411670  if (v79 == v80) goto VMPCrack.0047E2E8//跳
0047DD76  |. /06  vJmp_00411670  if (v40 == s43) goto VMPCrack.004318DE//跳
0048D8F3  |. /06  vJmp_00411670  if (Cross(nonentity, v111) == v121) goto VMPCrack.004A927A//跳
0047F698  |. /94  vJmp_00411670  if (entryVMEax_45CED7 != entryVMEsi_45CED7) goto VMPCrack.00467F03//不跳
004CAA21  |. /06  vJmp_00411670  if (v119 != v120) goto VMPCrack.00437E53//不跳
```

跳到返回0的必须不跳才能能验证成功，跳过返回0的必须跳。先调试一下，跟踪时用设置`vEIP`功能设置到正确的目标，这回弹出了“注册码正确，验证完成。”，看来是可以爆破的。但是为什么刚才内容地址是`1`呢，再回去看看那个`1`是哪里来的，向上跟踪数据来源，最后找到了：

```
004DFC32  |.  00 000000  vPushImm4 407470  (常量)407470
004DFC30  |.  6F         vReadMemDs4       内存00407470:(常量)1; 堆栈-0B4:(常量)407470
```

是从地址`407470`里读出来的，插件认为内存是常量，所以直接显示了，再看看`407470`的值，已经变成了`3420000`，应该是注册成功后写入的。到返回`1`的那个`vRet`往前看看，前面有：

```
00446EE6  |. /1B  vJmp_00411C30  callVM VMPCrack.0046BB01
004B16AB  |.  8F  vWriteMemDs4   DWORD DS:[407470] = v136; EXIT DWORD v136 = entryVMEax_4689BB
```

字符串地址是一个调用返回的，跟进那个调用：

```
0046BB01  /$  7B     vPopReg4 vR5
0046B9AB  |.  4C     vReadMemSs4     DWORD m0 = DWORD SS:[Je(AndFlag(0, 0)) + 28]
0046B987  |.  F4     vPushReg4 vR12  DWORD v0 = 0
0046B979  |.  2B     vJmp_00411C30   if (0 != 0) goto VMPCrack.004B8F91
004B9438  |.  71     vPopReg4 vR15
004B93CD  |.  0B     vAdd4           ARG4 DWORD v1 = 4
004B939A  |.  2C     vNand4          ARG3 DWORD v2 = 3000
004B9378  |.  66     vNand4          ARG2 DWORD v3 = 18
004B9340  |.  41     vAdd4           ARG1 DWORD v4 = 0
004B92C2  |.  17 04  vCall 4         v5 = Call(495AD3); VMPCrack.00495AD3	//VirtualAlloc
004B9250  |.  CE     vWriteMemDs4    DWORD DS:[v5] = v6; DWORD v6 = 0E1B2A2D7	//产生字符串
004B91EA  |.  16     vWriteMemDs4    DWORD DS:[v5 + 4] = v7; DWORD v7 = 0FDD5EBC2
004B913E  |.  E2     vWriteMemDs4    DWORD DS:[v5 + 8] = v8; DWORD v8 = 0ACA3B7C8
004B90AA  |.  16     vWriteMemDs4    DWORD DS:[v5 + 0C] = v9; DWORD v9 = 0A4D6E9D1
004B9057  |.  16     vWriteMemDs4    DWORD DS:[v5 + 10] = v10; DWORD v10 = 0C9B3EACD
004B902C  |.  16     vWriteMemDs4    DWORD DS:[v5 + 14] = v11; DWORD v11 = 0A3A1
004B8FE2  |.  E2     vWriteMemDs4    DWORD DS:[491D42] = v12; EXIT DWORD v12 = v5
004B8F9E  |.  F8     vPushReg4 vR8   DWORD v0 = v5
...
004B8C46  \.  86     vRet            return v15
```

有`vCall`，还是`SDK`，应该是解密字符串。上面有一个`if (0 != 0) goto VMPCrack.004B8F91`，0和0比较，一般不会这么做，最有可能的是其中一个是内存中读出来的，被插件当作常量了，往上看看，有：

```
0046BA92  |.  00 000000  vPushImm4 491D42  (常量)491D42
0046BA90  |.  08         vReadMemDs4       内存00491D42:(常量)0; 堆栈24:(常量)491D42
```

确实是内存里读出来的，加个分析提示告诉插件这里不是常量，`/*memConst(0,491D42,4)*/`表示`491D42`开始的4字节不是常量，一定要加到内存块开始，就是`40A000`。再分析一下，上面的比较变成了：

```
0046BA90  |.  08  vReadMemDs4     DWORD m0 = DWORD DS:[491D42]
0046B9AB  |.  4C  vReadMemSs4     DWORD m1 = DWORD SS:[Je(AndFlag(m0, m0)) + 28]
0046B987  |.  F4  vPushReg4 vR12  DWORD v0 = m0
0046B979  |.  2B  vJmp_00411C30   if (m0 != 0) goto VMPCrack.004B8F91
```

检查是否已经解密过字符串，如果解密过直接使用，否则调用`VirtualAlloc`分配内存，产生字符串保存起来。

现在可以爆破了，把那5个条件转移改掉就行，因为加了壳，这部分代码是压缩的，只能在解压后修改。你如果觉得脱壳更简单那就脱吧，我不会脱壳，连最简单的压缩壳都没有脱过，一直都是带壳分析带壳修改，这次为了测试插件才研究了一下VMP脱壳，后面会讲一下。先把补丁做好，方法很多，能改转移目标就行，比如把两个解密前的目标改成相同的值、在靠近`vJmp`的位置插入代码修改目标、在验证失败的指令块开始添加`vJmp`转到正确的目标等。因为加了最大保护，解密前的目标不好修改，这里先讲一下使用插入指令的方法。随便找一个分支，比如`0047DD76  |. /33  vJmp_00411670  if (v40 == s43) goto VMPCrack.004318DE`。

这个分支必须跳，由于插入指令至少需要8字节的空间来添加一些附加的代码，所以不能直接在`vJmp`插入，应该向前找几个指令，最近的是`0047DD80  |.  63 15CDF8FA  vPushImm4 0EF74CFD4  DWORD _t51643 = 0EF74CFD4`。

在这里选择插入指令，会弹出一个窗口，建好了插入指令的框架，内容是：

```
vPopReg4 vR14 ;弹出重定位
 ;在这里添加指令
vPushImm4 0EF74CFD4 ;被覆盖的指令
vPushReg4 vR3 ;被覆盖的指令
vAdd4 ;被覆盖的指令
vPopReg4 vR5 ;被覆盖的指令
vPushReg4 vR14 ;重定位
vPushImm4 0047DD79 ;目标地址
vJmp_00411670 ;转到原来的位置
```

在上面标出的位置添加指令，由于是想修改转移目标，看看保存在哪个寄存器中。执行`vJmp`时栈顶的值是返回地址，一般是离`vJmp`最近的那个`vPushReg4`压入的，这个寄存器是`vR9`，所以添加代码：

```
vPushImm4 4318DE
vPopReg4 vR9
```

这样就可以改变转移目标了，由于`VMP`中没有`MOV`指令，只能通过这种方法来赋值。然后找个地方来保存插入的代码，因为要带壳修改，最好放到不用解压的段里，一般找包含模块入口点的段就可以了，这个段在解压前就会执行。这个程序里是`.vmp1`，在后面找一块空的地方。选择保存位置时要注意是否为反向指令块，这时代码是从后向前保存的，这次要修改的代码就在反向指令块中，所以直接放在最后就行了，地址设置为`5C0FFF`。

插入指令会把原来的指令块拆分成两个，添加一个`vJmp`到插入的指令，执行插入代码后再执行被覆盖的指令，最后跳回原来的位置，还会自动处理重定位等。这是比较通用的方法，可以用于任何修改，而且有些修改由于会覆盖原来的指令，只能用这种方法，但缺点是生成的代码比较多。如果只是想改转移目标的话还有更简单的方法，就是在验证失败的指令块开始添加`vJmp`转到正确的目标，还是刚才那个分支，验证失败时转到`004C9665`，可以在这里修改代码，转到正确的目标`4318DE`：

```
vPushImm4 4318DE
vJmp_00411670
```

只修改这两条指令就可以了，但是因为`VMP`使用类似流加密的方法加密指令，即使只修改一个指令，也要重新编译从修改位置到指令块结束的所有代码，这些插件都自动做了。由于是在指令块开始修改，整个指令块都重新编译了，修改范围太大，最大保护下很容易碰到验证完整性的代码，还要处理验证。但是这里修改后的指令是`vJmp`，后面的指令不会执行了，打补丁时只写入这两条指令就可以。把其他几个转移也修改一下，下面是修改后的代码和对应的内存内容：

```
004400A8      8A 1914727C  vPushImm4 47E2E8
004400A3      64           vJmp_00411670
004400A2  64 19 14 72 7C 8A
//
004C9665      47 C8CD0790  vPushImm4 4318DE
004C9660      23           vJmp_00411670
004C965F  23 C8 CD 07 90 47
//
004C9B00      22 95AD0CE9  vPushImm4 4A927A
004C9AFB      A2           vJmp_00411670
004C9AFA  A2 95 AD 0C E9 22
//
00467F03      21 14AEF0B4  vPushImm4 43C25B
00467EFE      82           vJmp_00411670
00467EFD  82 14 AE F0 B4 21
//
00437E53      71 E3CBEF76  vPushImm4 461C3F
00437E4E      B6           vJmp_00411670
00437E4D  B6 E3 CB EF 76 71
```
