
# 一道 defcon 题的分析

> 原文：`www.pediy.com/kssd/pediy12/142985.html`

ps：第一次做 defcon 的题，不怎么了解老美比赛的思路，不知道题做到哪一步才是要求的结果

---

## 参考资料

1. `http://blog.csdn.net/yangbostar/article/details/6107169`
2. `http://hi.baidu.com/coderui/blog/item/681df92bedfeb4ffe7cd4060.html`
3. `http://fujiale.aau.cn/archiver/?tid-34.html`

**环境**：`Win7` 旗舰版  
**工具**：`IDA` & `OD` & `UE`

---

## 修改错误

`0x7E4507EA` 内存读写错误，怀疑是中文 `windows` 导致的问题，`google` 了下，发现是 `MessageBoxA()` 的地址，改为 `0x7748EA71`，保存运行，貌似是句俚语（见文章最后的图片）

---

## 分析

### 行为收集

没有释放 `dll`，没有进行双进程保护；创建了多个线程

### 代码分析

#### IDA 静态分析

**关键函数分析**：

- `sub_401220`：首先调用 `sub_401000` 得到 `DLL` 基地址，然后调用 `sub_401170` 得到一些 `API` 的入口地址。结合 `OD` 调试信息，可得出 `dll` 为 `kernel32.dll`,和 `ntdll.dll`。
- `sub_401000`：调用 `sub_401060` 返回 `PEB` 结构，进入 `PEB_LDR_DATA`，调用 `sub_401070`，通过比对 `dllname` 得到需要的 `DLL`，并返回 `baseaddress`。
- `sub_401170`：找到内存中 `DLL` 镜像的输入表，调用 `sub_4010f0` 比对 `name`，找到需要的 `API`，并返回内存地址。
- `sub_401930`：具体的线程执行函数

#### OD 动态分析

**程序流程**：

调用 `sub_401220` 得到 `API` 入口，然后使用 `InitializeCriticalSectionAndSpinCount` 得到 `debug` 信息，之后是四次循环，循环中创建线程 `sub_401930`，之后在 `7762639f` 处发生溢出。

**重点函数分析 `sub_401930`**：

##### 反调试

```
00401AEC   FFD1   CALL ECX;
```

将线程对调试器隐藏：`ZwSetInformationThread(GetCurrentThread(), ThreadHideFromDebugger, NULL, 0);`

```
00401AF5   . 8B40 68       MOV EAX,DWORD PTR DS:[EAX+68]            ;
```

`PEB` 结构中的 `NtGlobalFlag`，有调试器时为 `0x70`，无调试器时为 `0`

```
00401E69   . FF15 00204000 CALL DWORD PTR DS:[<&KERNEL32.CloseHandl>;
```

调用 `CloseHandle` 释放一个无效句柄，如果进程在调试器之外，那么函数返回 `FALSE`，而 `GetLastError` 得到 `ERROR_INVALID_HANDLE`；但是如果进程在调试器内，那么系统将抛出异常 `C0000008H`。

##### 修改反调试代码

1. `ZwSetInformationThread` 未修改，使用断点即可 调试

2. `NtGlobalFlag`：

   将 `JNZ SHORT b500_333.00401AFF` 改为 `JNZ SHORT b500_333.00401AFF`

3. `CloseHandle`：

   将：
   ```
   call    CloseHandle
   ```
   改为
   ```
   pop    eax

   此处由于代码逻辑是要执行 seh 处代码，

   JMP ESI

   NOP

   NOP

   NOP
   ```

   修改完后，执行时发现 堆栈溢出，所以修改堆栈

   ```
   00401E7C   . 90          NOP
   00401E7D   . 90          NOP
   00401E7E   . 90          NOP
   00401E7F   . 90          NOP
   00401E80   . 90          NOP
   00401E81   . 90          NOP
   00401E82   . 90          NOP
   ```

   修改之后，程序可以调试，但是有时会“跑飞”，需要经常下断点。

   分析到这里，大概猜到这道题的意思，可能是让 通过调试，找到最终需要的字符传？

   下面进行解密分析，基本的思路是通过线程同步解密 和 分段解密，造成一种迷惑效果，不过只要能调试，其他的都只是时间问题~

##### 字符串解密部分

1. ```
   00401C18   > /AC          LODS BYTE PTR DS:[ESI]
   00401C19   . |32C3        XOR AL,BL
   00401C1B   . |AA          STOS BYTE PTR ES:[EDI]
   00401C1C   . |49          DEC ECX
   00401C1D   . |E3 02       JECXZ SHORT b500_333.00401C21
   00401C1F   .^\EB F7       JMP SHORT b500_333.00401C18
   ```

2. ```
   00401BB5   . FC                  CLD
   00401BB6   . 8BB5 CCFCFFFF      MOV ESI,DWORD PTR SS:[EBP-334]
   00401BBC   . 8BBD CCFCFFFF      MOV EDI,DWORD PTR SS:[EBP-334]
   00401BC2   . 8B9D D8FCFFFF      MOV EBX,DWORD PTR SS:[EBP-328]
   00401BC8   . B9 00020000        MOV ECX,200
   00401BCD   . AC                 LODS BYTE PTR DS:[ESI]
   00401BCE   . 32C3               XOR AL,BL
   00401BD0   . AA                 STOS BYTE PTR ES:[EDI]
   ```

3. ```
   00401BD3   . FC                  CLD
   00401BD4   . 8BB5 CCFCFFFF      MOV ESI,DWORD PTR SS:[EBP-334]
   00401BDA   . 8BBD CCFCFFFF      MOV EDI,DWORD PTR SS:[EBP-334]
   00401BE0   . 8B9D D8FCFFFF      MOV EBX,DWORD PTR SS:[EBP-328]
   00401BE6   . B9 00020000        MOV ECX,200
   00401BEB   . AC                 LODS BYTE PTR DS:[ESI]
   00401BEC   . 32C3               XOR AL,BL
   00401BEE   . AA                 STOS BYTE PTR ES:[EDI]
   ```

4. `sub_00401F10`

5. ```
   003C0008   /EB 75               JMP SHORT 003C007F
   003C000A   |55                  PUSH EBP
   003C000B   |8BEC               MOV EBP,ESP
   003C000D   |83EC 20            SUB ESP,20
   003C0010   |8B5D 04            MOV EBX,DWORD PTR SS:[EBP+4]
   003C0013   |B9 08000000        MOV ECX,8
   003C0018   |49                 DEC ECX
   003C0019   |FF348A             PUSH DWORD PTR DS:[EDX+ECX*4]
   003C001C   |89048A             MOV DWORD PTR DS:[EDX+ECX*4],EAX
   003C001F   |85C9               TEST ECX,ECX
   003C0021  ^|77 F5             JA SHORT 003C0018
   003C0023   |8BD4               MOV EDX,ESP
   003C0025   |8D73 08            LEA ESI,DWORD PTR DS:[EBX+8]
   003C0028   |8D7B 08            LEA EDI,DWORD PTR DS:[EBX+8]
   003C002B   |FC                 CLD
   003C002C   |B9 C0030000        MOV ECX,3C0
   003C0031   |33DB               XOR EBX,EBX
   003C0033   |66:AD              LODS WORD PTR DS:[ESI]
   003C0035   |66:33C1            XOR AX,CX
   003C0038   |66:33045A          XOR AX,WORD PTR DS:[EDX+EBX*2]
   003C003C   |66:AB              STOS WORD PTR ES:[EDI]
   003C003E   |49                 DEC ECX
   003C003F   |85C9               TEST ECX,ECX
   003C0041   |74 0A              JE SHORT 003C004D
   003C0043   |43                 INC EBX
   003C0044   |83FB 10            CMP EBX,10
   003C0047  ^|72 EA             JB SHORT 003C0033
   003C0049   |33DB               XOR EBX,EBX
   003C004B  ^|EB E6             JMP SHORT 003C0033
   003C004D   |0F31               RDTSC
   003C004F   |8BD1               MOV EDX,ECX
   003C0051   |BB 14000000        MOV EBX,14
   003C0056   |F7F3               DIV EBX
   003C0058   |8BC2               MOV EAX,EDX
   003C005A   |BB 60000000        MOV EBX,60
   003C005F   |F7E3               MUL EBX
   003C0061   |8B5D 04            MOV EBX,DWORD PTR SS:[EBP+4]
   003C0064   |8D5403 08          LEA EDX,DWORD PTR DS:[EBX+EAX+8]
   003C0068   |33C0               XOR EAX,EAX
   003C006A   |50                 PUSH EAX
   003C006B   |68 4D4F544D        PUSH 44544F4D
   003C0070   |8BF4               MOV ESI,ESP
   003C0072   |50                 PUSH EAX
   003C0073   |56                 PUSH ESI
   003C0074   |52                 PUSH EDX
   003C0075   |50                 PUSH EAX
   003C0076   |FF13               CALL DWORD PTR DS:[EBX]   MessageboxA
   003C0078   |83C4 04            ADD ESP,4
   003C007B   |50                 PUSH EAX
   003C007C   |FF53 04            CALL DWORD PTR DS:[EBX+4]
   003C007F   \E8 86FFFFFF        CALL 003C000A
   ```

---

## 结果

会出现的字符串如下：

```
001F0080                               49 6E 20 74          t
001F0090   68 65 20 62 65 67 69 6E 6E 65 72 27 73 20 6D 69  he beginner's mi
001F00A0   6E 64 20 74 68 65 72 65 20 61 72 65 20 6D 61 6E  nd there are man
001F00B0   79 20 70 6F 73 73 69 62 69 6C 69 74 69 65 73 2C  y possibilities,
001F00C0   20 62 75 74 20 69 6E 20 74 68 65 20 65 78 70 65   but in the expe
001F00D0   72 74 27 73 20 6D 69 6E 64 20 74 68 65 72 65 20  rt's mind there
001F00E0   61 72 65 20 66 65 77 2E 00 00 00 00 4E 6F 20 73  are few.....No s
001F00F0   6E 6F 77 66 6C 61 6B 65 20 65 76 65 72 20 66 61  nowflake ever fa
001F0100   6C 6C 73 20 69 6E 20 74 68 65 20 77 72 6F 6E 67  lls in the wrong
001F0110   20 70 6C 61 63 65 2E 00 00 00 00 00 00 00 00 00   place..........
001F0120   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
001F0130   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
```
