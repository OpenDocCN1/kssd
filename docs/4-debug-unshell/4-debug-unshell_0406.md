
# 斑斑传奇主程序脱壳

> 原文：[`www.pediy.com/kssd/pediy06/pediy6655.htm`](https://www.pediy.com/kssd/pediy06/pediy6655.htm)

目标程序：[`bbs.pediy.com/upload/bbs/bbcq.rar`](http://bbs.pediy.com/upload/bbs/bbcq.rar)

**第一部分: 加密调用程序，ASPR1.2x，无需脱壳**

***其实无需分析，闲来无事，就看了看![](img/620448a77d884bd6b2c4a7cfef437501.jpg)

运行并挂起被加密的程序:

```
00122F98   0040145F  /CALL to CreateProcessA
00122F9C   00000000  |ModuleFileName = NULL
00122FA0   0012304C  |CommandLine = "E:\Documents and Settings\Star\",D7,"烂鎈 1088229761\Bbcq.DLL"
00122FA4   00000000  |pProcessSecurity = NULL
00122FA8   00000000  |pThreadSecurity = NULL
00122FAC   00000000  |InheritHandles = FALSE
00122FB0   00000004  |CreationFlags = CREATE_SUSPENDED
00122FB4   00000000  |pEnvironment = NULL
00122FB8   00000000  |CurrentDir = NULL
00122FBC   00123008  |pStartupInfo = 00123008
00122FC0   00122FF0  \pProcessInfo = 00122FF0
```

分配内存作解码缓冲:

```
00122FB8   004014FB  /CALL to GlobalAlloc from bbcqrun.004014F9
00122FBC   00000000  |Flags = GMEM_FIXED
00122FC0   00026000  \MemSize = 26000 (155648.)
```

***我这里返回 `00148420`,一会注意

修正页面访问权限:

```
00122FAC   00401575  /CALL to VirtualProtectEx from bbcqrun.00401573
00122FB0   00000050  |hProcess = 00000050 (window)
00122FB4   00401000  |Address = bbcqrun.<ModuleEntryPoint>
00122FB8   00013000  |Size = 13000 (77824.)
00122FBC   00000004  |NewProtect = PAGE_READWRITE
00122FC0   00122FEC  \pOldProtect = 00122FEC
```

读出需要解密的数据:

```
00122FAC   004015E8  /CALL to ReadProcessMemory from bbcqrun.004015E6
00122FB0   00000050  |hProcess = 00000050 (window)
00122FB4   00401000  |pBaseAddress = 401000
00122FB8   00148420  |Buffer = 00148420
00122FBC   00013000  |BytesToRead = 13000 (77824.)
00122FC0   00123000  \pBytesRead = 00123000
```

***可以看出 `401000` 处的 `13000` (可能比这个值小) 需要处理

执行解码算法:

分析寄存器数值-->

`EDI` == `00148420` (缓冲区)

`ECX` -> `123000` (实际读出字节)

`EDX` == `13000` (传递的大小)

分析解码算法-->

```
004015E8   .  B8 04000000   MOV     EAX, 4                   ; eax = 4
004015ED   .  C74424 14 000>MOV     DWORD PTR [ESP+14], 0        ; 清一个变量,设为 i
004015F5   .  2BC7          SUB     EAX, EDI                 ; eax = eax - edi = 4 - buf = -1344540
004015F7   .  8BCF          MOV     ECX, EDI                 ; ecx -> 缓冲区
004015F9   .  894424 1C     MOV     [ESP+1C], EAX            ; 一个变量 = eax ,设为 k
004015FD   .  EB 04         JMP     SHORT 00401603
```

*** `k` 的作用是调整指针

```
004015FF   >  8B4424 1C     MOV     EAX, [ESP+1C]            ; eax = k
00401603   >  03C1          ADD     EAX, ECX                 ; eax = eax + ecx
00401605   .  3D 00300100   CMP     EAX, 13000               ; 到达数据最大长度
0040160A   .  77 28         JA      SHORT 00401634           ; 解码结束
0040160C   .  8B01          MOV     EAX, [ECX]               ; 读出 4 字节用作解密
0040160E   .  83C1 04       ADD     ECX, 4                   ; 指针后挪
00401611   .  8BD0          MOV     EDX, EAX                 ; edx = eax
00401613   .  F7D2          NOT     EDX
00401615   .  33D0          XOR     EDX, EAX
00401617   .  81E2 5A5A3C3C AND     EDX, 3C3C5A5A
0040161D   .  F7D0          NOT     EAX
0040161F   .  33D0          XOR     EDX, EAX
00401621   .  8B4424 14     MOV     EAX, [ESP+14]            ; eax = i
00401625   .  8951 FC       MOV     [ECX-4], EDX             ;  将运算后的数据写回去
00401628   .  40            INC     EAX                      ; eax++
00401629   .  3D 00010000   CMP     EAX, 100                 ; 是否解码完成
0040162E   .  894424 14     MOV     [ESP+14], EAX            ; eax 写回 i
```

*** `i` 的作用是控制循环

```
00401632   .^ 72 CB         JB      SHORT 004015FF           ;  循环解码
```

*** 由上述分析得到需要解码信息-->

地址 `401000h` 长度 `100h` \* 4 = `400h`

算法 (略)

将解码后的数据写回原程序:

```
00122FAC   004016D6  /CALL to WriteProcessMemory from bbcqrun.004016D4
00122FB0   00000050  |hProcess = 00000050 (window)
00122FB4   00401000  |Address = 401000
00122FB8   0015B420  |Buffer = 0015B420
00122FBC   00013000  |BytesToWrite = 13000 (77824.)
00122FC0   00123000  \pBytesWritten = 00123000
```

改变访问权限:

```
00122FAC   004016F3  /CALL to VirtualProtectEx from bbcqrun.004016EF
00122FB0   00000050  |hProcess = 00000050 (window)
00122FB4   00401000  |Address = bbcqrun.<ModuleEntryPoint>
00122FB8   00013000  |Size = 13000 (77824.)
00122FBC   00000080  |NewProtect = PAGE_EXECUTE_WRITECOPY
00122FC0   00122FEC  \pOldProtect = 00122FEC
```

释放缓冲:

```
00122FBC   00401752  /CALL to GlobalFree from bbcqrun.00401750
00122FC0   00148420  \hMem = 00148420
```

激活被挂起的线程:

```
00122FBC   004017AF  /CALL to ResumeThread from bbcqrun.004017AD
00122FC0   00000054  \hThread = 00000054 (window)
```

现在这个 `Loader` 的任务完成,他无奈的自杀了-_-!

```
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
```

**第二部分: `bbcq.dll`, DBPE 2.x, 目的脱壳**

这是一个伪 DLL,实际是 EXE.

似乎不好直接 Dump...我们来一个取巧的方法.

`OllyDbg` 载入 `bbcq.dll`，在入口写入数据 `eb fe(jmp eip)`,保存一下

顺便记住原来的指令 `EB 20`

运行 `bbcqrun.exe`,没反应?进程已经挂起了，赶快 dump...保存为 `dumped.exe`,关掉 dll 进程

我们坚持使用脱壳机...毕竟幻影不是很好惹，有多少爱可以重来？

把入口指令改回去，入口也改回去，`UnDBPE` 伺候![](img/620448a77d884bd6b2c4a7cfef437501.jpg)

选上优化...最后用 `LordPE` `Rebuild` 一下

完美脱壳

`4dd466d0e1e0cc123afbe504da2462ca`
