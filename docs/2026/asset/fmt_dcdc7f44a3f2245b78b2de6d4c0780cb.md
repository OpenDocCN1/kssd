
```
0043183F  |.  93  vReadMemSs4     DWORD v82 = GetBytes(MD5, 0, 4)
004DA066  |.  A4  vReadMemSs4     DWORD v83 = GetBytes(MD5, 0C, 4)
004D9FDD  |.  2B  vReadMemSs4     DWORD v84 = GetBytes(MD5, 8, 4)
0042B0C3  |.  49  vWriteMemSs1    EXIT BYTE v85 = (GetBytes(v51, 0, 1) ^ v46) + (GetBytes(MD5, 0, 1) ^ 0AA)
0042AF7A  |.  2B  vReadMemSs4     DWORD v86 = GetBytes(MD5, 4, 4)
004AB52C  |.  D3  vWriteMemSs1    EXIT BYTE v87 = (GetBytes(v51, 1, 1) ^ v48) + (GetBytes(MD5, 5, 1) ^ 0BB)
004B20AF  |.  38  vWriteMemSs1    EXIT BYTE v88 = (v49 ^ GetBytes(v51, 2, 1)) + (GetBytes(MD5, 0A, 1) ^ 0CC)
00482E81  |.  38  vWriteMemSs1    EXIT BYTE v89 = (GetBytes(MD5, 0F, 1) ^ 0DD) + (GetBytes(v51, 3, 1) ^ v50)
0045F1B8  |.  E1  vDiv4           DWORD v90 = 0 : (v82 ^ (v83 ^ v84 ^ v86)) % 5
004D277E  |.  2B  vReadMemSs4     DWORD m27 = DWORD SS:[Ja(SubFlag(v90, 4)) + 0FFFFFFE8]
004D275E  |.  19  vPushReg4 vR11  DWORD v91 = v84
004D274B  |.  94  vJmp_00411670   if (v90 > 4) goto VMPCrack.00470963
0047C94E  |.  8C  vReadMemDs4     DWORD m28 = DWORD DS:[(v90 << 2) + 40609C]
0047C920  |.- 06  vJmp_00411670   switch (v90)
004DD0F5  |>  F3  vPopReg4 vR2								//switch 0
00437358  |.  31  vWriteMemSs4    EXIT DWORD v92 = (v82 ^ 11223344) + (v86 ^ 22334455)
004B3409  |>  03  vPopReg4 vR6								//switch 1
004B31D6  |.  C7  vNand4          DWORD v93 = v91 ^ 44556677
0048CF6D  |.  31  vWriteMemSs4    EXIT DWORD v92 = v93 + (v86 ^ 33445566)
0048CD9F  |.  1D  vPushReg4 vR12  DWORD v91 = v93
0047C627  |>  FB  vPopReg4 vR4								//switch 2
0049ED8F  |.  00  vNand4          DWORD v94 = v91 ^ 55667788
004E19BC  |.  31  vWriteMemSs4    EXIT DWORD v92 = v94 + (v83 ^ 66778899)
004E18D6  |.  F1  vPushReg4 vR1   DWORD v91 = v94
004A9743  |>  EF  vPopReg4 vR1								//switch 3
00472BED  |.  31  vWriteMemSs4    EXIT DWORD v92 = (v83 ^ 778899AA) + (v82 ^ 8899AABB)
0045E0BA  |>  0B  vPopReg4 vR8								//switch 4
0045DF80  |.  00  vNand4          DWORD v95 = v91 ^ 0AABBCCDD
0045DE90  |.  00  vNand4          DWORD v96 = v82 ^ 99AABBCC
00470B2F  |.  48  vAdd4           DWORD v97 = v96 + v95
004709E8  |.  BA  vWriteMemSs4    EXIT DWORD v92 = v97
0047098C  |.  FD  vPushReg4 vR4   DWORD v91 = v97
00470963  |>  1B  vPopReg4 vR12
0047881C  |.  31  vWriteMemSs4    "结果" EXIT DWORD v98 = 14
0047CAB2  |.  31  vWriteMemSs4    "密码" EXIT DWORD v99 = 0FFFFFFFC			//解密后的MD5
00434420  |.  BA  vWriteMemSs4    "内容" EXIT DWORD v100 = 14				//上面计算的内容
00481A39  |.  11  vPushReg4 vR9   EXIT DWORD v101 = v91
00481A32  |.  6F  vAdd4           EXIT DWORD v102 = 0EF8B1AAC
00481A2E  |.  52  vJmp_00412430   callVM <VMPCrack.TEA解密>
004B4216  |.  2B  vReadMemSs4     DWORD v103 = Cross(GetBytes("注册码", 8, 4), nonentity)
004B4154  |.  A4  vReadMemSs4     DWORD v104 = v89 : (v88 : (v87 : v85))
004B3F46  |.  00  vNand4          DWORD v105 = v103 ^ 13579BDF
004CAA54  |.  CE  vReadMemSs4     DWORD m29 = DWORD SS:[Je(SubFlag(v104, v105)) + 0FFFFFFE8]
004CAA21  |.  06  vJmp_00411670   if (v104 != v105) goto VMPCrack.00437E53		//检查注册码第三部分
0046198A  |.  A4  vReadMemSs4     DWORD v106 = Cross(GetBytes("注册码", 0C, 4), nonentity)
004A8CA3  |.  73  vNand4          DWORD v107 = v106 ^ 0FDB97531
0048D926  |.  2B  vReadMemSs4     DWORD m30 = DWORD SS:[Jnz(SubFlag(Cross(nonentity, v92), v107)) + 0FFFFFFE8]
0048D8F3  |.  06  vJmp_00411670   if (Cross(nonentity, v92) == v107) goto VMPCrack.004A927A//检查注册码第四部分
0049E1D7  |.  2B  vReadMemSs4     DWORD s4 = Stack(vESP + 34, ESP + 0, 4)
...
0049DE75  |.  2E  vRet            return v110
```

这部分因为有一个`switch`，可能乱一点。一开始是把`MD5`的4个`DWORD`做异或，然后除以5取余数作为`switch`索引选择一个分支，每个分支有不同的算法。索引对应的分支可以看看`switch`的注释，第一个目标是0，第二个是1，依此类推。后面又是一个调用，是`tea`解密，代码和加密很像，也是前后加了异或，就不贴出来了。下面是两个比较，分别检查注册码的第三和第四个`DWORD`，算法应该很容易看出来了。这次不是直接和解密后的注册码比较，而是先把注册码异或了一个常量。第二个比较有一个`Cross(nonentity, v92)`，也是因为`switch`前有一个分支跳了过去，这个路径没有变量定义。

```
004C04FE  |.  BA  vWriteMemSs4    "长度" EXIT DWORD v119 = 14
004574ED  |.  31  vWriteMemSs4    "内容" EXIT DWORD v120 = 20	//输入的注册码
00446FD7  |.  15  vPushReg4 vR10  EXIT DWORD v121 = XorFlag(Cross(GetBytes("注册码", 4, 4), nonentity) + Cross(GetBytes("注册码", 0, 4), nonentity), v103 + v106)
00446FD0  |.  6F  vAdd4           EXIT DWORD v122 = 0EF8B1AAC
00446FCC  |.  45  vJmp_00411C30   callVM <VMPCrack.CRC>
0047F6CA  |.  CE  vReadMemSs4     DWORD m35 = DWORD SS:[Je(SubFlag(entryVMEax_45CED7, entryVMEsi_45CED7)) + 0FFFFFFE8]
0047F698  |.  94  vJmp_00411670   if (entryVMEax_45CED7 != entryVMEsi_45CED7) goto VMPCrack.00467F03
00446EEA  |.  AD  vAdd4           EXIT DWORD v123 = 0EF8B1AAC
00446EE6  |.  1B  vJmp_00411C30   callVM VMPCrack.0046BB01	//调用SDK，解密字符串
004B16AB  |.  8F  vWriteMemDs4    DWORD DS:[407470] = v124; EXIT DWORD v124 = entryVMEax_4689BB
004A259F  |.  93  vReadMemSs4     DWORD s5 = Stack(vESP + 34, ESP + 0, 4)
...
004A22A1  |.  2E  vRet            return v127
```

开始是一个调用，跟进去看看，第一个转移是垃圾分支。

```
0042C409  |. /2B  vJmp_00411C30  if (Ja(1 ^ unknownInit2)) goto VMPCrack.004C9F2B
```

添加提示`/*jmp(4C9F2B)*/`清除掉，重新反汇编。

```
0042C5FB  /$  7A  vPopReg4 vR6
004C9CCE  |.  19  vReadMemSs4      DWORD s0 = Stack(vESP + 3C, ESP + 8, 4)	//参数2
004C83CD  |.  19  vReadMemSs4      DWORD m1 = DWORD SS:[Jle(AndFlag(s0, s0)) + 28]
004C83AB  |.  F2  vPushReg4 vR14   DWORD v0 = s0
004C83AA  |.  F4  vPushReg4 vR12   DWORD v1 = unknownInit10
004C83A9  |.  F5  vPushReg4 vR11   DWORD v2 = unknownInit4
004C83A7  |.  FC  vPushReg4 vR4    DWORD v3 = unknownInit9 | 0FFFFFFFF
004C83A5  |.  F7  vPushReg4 vR9    DWORD v4 = 0
004C839A  |.  2B  vJmp_00411C30    if (s0 <= 0) goto VMPCrack.0046D138
004ACA12  |.  B6  vReadMemSs4      DWORD s1 = Stack(vESP + 38, ESP + 4, 4)	//参数1
004AC8C9  |.  F3  vPushReg4 vR13   DWORD v5 = s1
004AC8BD  |>  77  /vPopReg4 vR9
004AC6D5  |.  FE  |vPushReg4 vR2   DWORD v6 = 0 : ((GetBytes(v3, 1, 1) ^ 0) + 1)
0048F74F  |.  20  |vReadMemDs1     BYTE m2 = BYTE DS:[v5 + v4]
004751ED  |.  AF  |vReadMemDs4     DWORD m3 = DWORD DS:[((v6 : m2 ^ v3 & 0FF) << 2) + 407030]
004A7C18  |.  4C  |vReadMemSs4     DWORD m4 = DWORD SS:[Jge(SubFlag(v4 + 1, v0)) + 20]
004A7BF8  |.  F9  |vPushReg4 vR7   DWORD v3 = m3 ^ v3 >> 8
004A7BF7  |.  FA  |vPushReg4 vR6   DWORD v4 = v4 + 1
004A7BF4  |.  F6  |vPushReg4 vR10  DWORD v1 = m3
004A7BE5  |.^ 1B  \vJmp_00411C30   if (v4 < v0) goto VMPCrack.004AC8BD
0046D138  |>  7F  vPopReg4 vR1
0046CBCC  |.  19  vReadMemSs4      DWORD s2 = Stack(vESP + 34, ESP + 0, 4)
...
0046C844  |.  F3  vPushReg4 vR13   EAX DWORD v12 = v3 ^ 6789ABCD
...
0046C83B  \.  86  vRet             return v9
```

先检查第二个参数是否大于0，然后是一个循环，次数由第二个参数控制，第一个参数是指针，这里可以看出来第二个参数就是内容长度。大部分计算都很明显，但是有一个`DWORD DS:[((v6 : m2 ^ v3 & 0FF) << 2) + 407030]`，`v6`长度是4字节，又连接上一个1字节的`m2`是5字节？`v6`的高3字节是0，我觉得可能是`0:m2`，这里的分析有点问题。循环里的代码连起来是

`DWORD DS:[((BYTE DS:[v5 + v4] ^ v3 & 0FF) << 2) + 407030] ^ v3 >> 8`

好像是`crc`，看看`407030`的内容。

```
00407030  00 00 00 00 96 30 07 77 2C 61 0E EE BA 51 09 99
```

很明显是`crc`使用的表了，刚才`peid`的`Krypto ANALyzer`插件显示的`crc`就是这个地址，但还是有点不一样，`crc`结果最后应该取反，这里是异或`6789ABCD`，可能又是修改过的算法，异或`6789ABCD`相当于取反后再异或`98765432`。填上标签和参数提示再回去看刚才的代码。

然后是一个比较`if (entryVMEax_45CED7 != entryVMEsi_45CED7)`，`eax`是调用`crc`的返回值，`esi`是什么？给`crc`函数加上初始寄存器提示吧，`/*initReg(esi,0C)*/`。调用前还有一个

`v121 = XorFlag(Cross(GetBytes("注册码", 4, 4), nonentity) + Cross(GetBytes("注册码", 0, 4), nonentity), v103 + v106)`

这是异或的标志，但是前面没见过这个计算，调用前计算的，很可能是调用需要的内容，没准是要计算`crc`的部分，但是插件不能自动分析被调用函数，不知道他引用了这个数据，所以认为是无效的了。给调用`crc`的`vJmp`加上一个`/*exitValid(1)*/`提示，表示调用退出虚拟机时可访问的数据都是有效的，再重新分析一下试试，这回变成了
