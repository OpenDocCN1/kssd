

# 手脱 Armadillo 5.0 (Standard + Debug-Blocker).学习笔记.

> 原文：[`www.pediy.com/kssd/pediy10/92699.html`](https://www.pediy.com/kssd/pediy10/92699.html)

【文章标题】: 手脱 Armadillo 5.0 (Standard + Debug-Blocker).学习笔记.  
【文章作者】: wuhanqi  
【作者邮箱】: wuhanqi@qq.com  
【软件名称】: 一个 unpackme  
【下载地址】: 自己搜索下载  
【加壳方式】: Armadillo 5.0 (Standard + Debug-Blocker)  
【编写语言】: Delphi  
【使用工具】: OD.ImportREC.LordPE.Arma Detach.PEID 0.94.FastScanner 2.1  
【操作平台】: 盗版 XP3  
【作者声明】: 小菜一个.高手别笑话我哦~嘿嘿.  
\--------------------------------------------------------------------------------  
【详细过程】  
  废话不多说.进入正题.脱壳目标为一个 unpackme.  
    
  FastScan 2.1 查壳为 Microsoft Visual C++ 8  
  PEID 0.94 查壳为 Armadillo V5.0X -> Silicon Realms Toolworks   \* Sign.By.fly \*  
    
  首先运行 Armadillo Process Detach.将 unpackme 脱入 Arma Detach 中。  
  会显示内容如下:  
  DONE!  
  Child process ID: 000002C8  
  Entry point: 004A2DC2  
  Original bytes: E8E3  
    
  这时我们打开一个 od.文件--附加.然后选中 PID 为 2C8 的进程.即为 Armadillo 的子进程.  
  我们载入后.F9.再 F12.便断到如下代码处:  
  

代码:

```
  004A2DC2 >- EB FE           JMP SHORT <ModuleEntryPoint>
  004A2DC4    40              INC EAX
  004A2DC5    0000            ADD BYTE PTR DS:\[EAX\],AL
  004A2DC7  ^ E9 16FEFFFF     JMP 004A2BE2
  004A2DCC    6A 0C           PUSH 0C
  004A2DCE    68 B0104D00     PUSH 004D10B0
  004A2DD3    E8 44150000     CALL 004A431C
  004A2DD8    8B4D 08         MOV ECX,DWORD PTR SS:\[EBP+8\]
  004A2DDB    33FF            XOR EDI,EDI
  004A2DDD    3BCF            CMP ECX,EDI
  004A2DDF    76 2E           JBE SHORT 004A2E0F
  004A2DE1    6A E0           PUSH -20
  004A2DE3    58              POP EAX
  004A2DE4    33D2            XOR EDX,EDX
  004A2DE6    F7F1            DIV ECX
  004A2DE8    3B45 0C         CMP EAX,DWORD PTR SS:\[EBP+C\]
  

```

  
  入口点是一个跳转.其实是 Arma Detach 为了防止穿山甲检测.才修改了入口点.  
  我们根据 Arma Detach 给我们的数据:Original bytes: E8E3.将 jmp 那两个字节修改为 E8E3.即可恢复正常的入口点.  
代码:

```
  004A2DC2 >  E8 E3400000     CALL 004A6EAA
  004A2DC7  ^ E9 16FEFFFF     JMP 004A2BE2
  004A2DCC    6A 0C           PUSH 0C
  004A2DCE    68 B0104D00     PUSH 004D10B0
  004A2DD3    E8 44150000     CALL 004A431C
  004A2DD8    8B4D 08         MOV ECX,DWORD PTR SS:\[EBP+8\]
  004A2DDB    33FF            XOR EDI,EDI
  004A2DDD    3BCF            CMP ECX,EDI
  004A2DDF    76 2E           JBE SHORT 004A2E0F
  004A2DE1    6A E0           PUSH -20
  004A2DE3    58              POP EAX
  004A2DE4    33D2            XOR EDX,EDX
  004A2DE6    F7F1            DIV ECX
  004A2DE8    3B45 0C         CMP EAX,DWORD PTR SS:\[EBP+C\]
  

```

  
  接下来我们需要确认 OD 没有硬件断点.以及 Alt+B 确认没有任何切换断点.  
  然后我们下 bp VirtualProtect.可以使我们来到穿山甲处理 IAT 的地址.  
  shift+f9 运行程序.我们要注意堆栈的变化:  
代码:

```
  0012F648   0048E8F7  /CALL to VirtualProtect from UnpackMe.0048E8F1
  0012F64C   00EB1000  |Address = 00EB1000
  0012F650   0006C82C  |Size = 6C82C (444460.)
  0012F654   00000040  |NewProtect = PAGE\_EXECUTE\_READWRITE
  0012F658   0012F67C  \\pOldProtect = 0012F67C
  
  0012F648   0048E8F7  /CALL to VirtualProtect from UnpackMe.0048E8F1
  0012F64C   00F1E000  |Address = 00F1E000
  0012F650   000088F4  |Size = 88F4 (35060.)
  0012F654   00000002  |NewProtect = PAGE\_READONLY
  0012F658   0012F67C  \\pOldProtect = 0012F67C
  
  0012F648   0048E8F7  /CALL to VirtualProtect from UnpackMe.0048E8F1
  0012F64C   00F27000  |Address = 00F27000
  0012F650   00012AC4  |Size = 12AC4 (76484.)
  0012F654   00000004  |NewProtect = PAGE\_READWRITE
  0012F658   0012F67C  \\pOldProtect = 0012F67C
  
  0012F648   0048E8F7  /CALL to VirtualProtect from UnpackMe.0048E8F1
  0012F64C   00F3A000  |Address = 00F3A000
  0012F650   000024C0  |Size = 24C0 (9408.)
  0012F654   00000002  |NewProtect = PAGE\_READONLY
  0012F658   0012F67C  \\pOldProtect = 0012F67C
  
  0012F648   0048E8F7  /CALL to VirtualProtect from UnpackMe.0048E8F1
  0012F64C   00F3D000  |Address = 00F3D000
  0012F650   00006464  |Size = 6464 (25700.)
  0012F654   00000002  |NewProtect = PAGE\_READONLY
  0012F658   0012F67C  \\pOldProtect = 0012F67C
  
  00129470   00EF9946  /CALL to VirtualProtect from 00EF9940
  00129474   00401000  |Address = UnpackMe.00401000
  00129478   0005B000  |Size = 5B000 (372736.)
  0012947C   00000004  |NewProtect = PAGE\_READWRITE
  00129480   0012C164  \\pOldProtect = 0012C164
  
  00129470   00EF9991  /CALL to VirtualProtect from 00EF998B
  00129474   00401000  |Address = UnpackMe.00401000
  00129478   0005B000  |Size = 5B000 (372736.)
  0012947C   00000020  |NewProtect = PAGE\_EXECUTE\_READ
  00129480   0012C164  \\pOldProtect = 0012C164
  
  00129470   00EF9946  /CALL to VirtualProtect from 00EF9940
  00129474   0045C000  |Address = UnpackMe.0045C000
  00129478   00002000  |Size = 2000 (8192.)
  0012947C   00000004  |NewProtect = PAGE\_READWRITE
  00129480   0012C164  \\pOldProtect = 0012C164
  
  00129470   00EF9991  /CALL to VirtualProtect from 00EF998B
  00129474   0045C000  |Address = UnpackMe.0045C000
  00129478   00002000  |Size = 2000 (8192.)
  0012947C   00000004  |NewProtect = PAGE\_READWRITE
  00129480   0012C164  \\pOldProtect = 0012C164
  
  00129470   00EF9946  /CALL to VirtualProtect from 00EF9940
  00129474   0045F000  |Address = UnpackMe.0045F000
  00129478   00003000  |Size = 3000 (12288.)
  0012947C   00000004  |NewProtect = PAGE\_READWRITE
  00129480   0012C164  \\pOldProtect = 0012C164
  
  00129470   00EF9991  /CALL to VirtualProtect from 00EF998B
  00129474   0045F000  |Address = UnpackMe.0045F000
  00129478   00003000  |Size = 3000 (12288.)
  0012947C   00000004  |NewProtect = PAGE\_READWRITE
  00129480   0012C164  \\pOldProtect = 0012C164
  
  00129470   00EFA814  /CALL to VirtualProtect from 00EFA80E
  00129474   0045F118  |Address = UnpackMe.0045F118
  00129478   0000008C  |Size = 8C (140.)    <<--我们一直 shift+f9.直到 size 显示为 8c.这样.我们就到了关键处.
  0012947C   00000004  |NewProtect = PAGE\_READWRITE
  00129480   0012C028  \\pOldProtect = 0012C028
  

```

  
  我们需要注意的是.在此之前断下来的 VM 都是 EXE 在执行 IAT 的处理.我们所需要断下来的地方是开始读写 IAT 的地方.  
  为什么我们需要当 size 为 8c 的时候返回呢?这里 size 8c 所代表的是 1000 个字节以上的意思.  
  即.如果我们的应用程序大小超过了 1000 个字节.那么便会显示 8c.如果超过了 2000 个字节.那么便会显示 16c.以此类推...  
  (我也不知道我解释清楚没...哎.语文功底不好...)  
  Let's go on!  
  此时我们 Ctrl+f9.或 F8.回到 VM 处!!  
  我们看下回到了哪里...  
代码:

```
  00EFA814    6A 14           PUSH 14                            <<--这里有像之前穿山甲版本一样的 push 14.
  00EFA816    E8 FBCA0000     CALL 00F07316
  00EFA81B    83C4 04         ADD ESP,4
  00EFA81E    8985 84ABFFFF   MOV DWORD PTR SS:\[EBP+FFFFAB84\],EAX
  00EFA824    83BD 84ABFFFF 0>CMP DWORD PTR SS:\[EBP+FFFFAB84\],0
  00EFA82B    74 59           JE SHORT 00EFA886
  00EFA82D    8B0D EC53F300   MOV ECX,DWORD PTR DS:\[F353EC\]
  00EFA833    898D 10AAFFFF   MOV DWORD PTR SS:\[EBP+FFFFAA10\],ECX
  00EFA839    8B95 74D8FFFF   MOV EDX,DWORD PTR SS:\[EBP-278C\]
  00EFA83F    0395 78D3FFFF   ADD EDX,DWORD PTR SS:\[EBP-2C88\]
  00EFA845    8B85 84ABFFFF   MOV EAX,DWORD PTR SS:\[EBP+FFFFAB84\]
  00EFA84B    8910            MOV DWORD PTR DS:\[EAX\],EDX
  

```

  
  哈哈.既然又像以前一样了.那我们也就好办了.  
  现在我们需要寻找 push 100.这条命令.重点是这条命令下面的 call 的地址.call 所调用的地址就是移动处理 IAT 的地址!  
  我们把滚轮向下滚...来到这里:  
代码:

```
  00EFAA7B    8B85 40C2FFFF   MOV EAX,DWORD PTR SS:\[EBP-3DC0\]
  00EFAA81    8378 08 00      CMP DWORD PTR DS:\[EAX+8\],0
  00EFAA85    74 4A           JE SHORT 00EFAAD1
  00EFAA87    68 00010000     PUSH 100                   <<---我们要找的 PUSH 100
  00EFAA8C    8D8D 40C1FFFF   LEA ECX,DWORD PTR SS:\[EBP-3EC0\]
  00EFAA92    51              PUSH ECX
  00EFAA93    8B95 40C2FFFF   MOV EDX,DWORD PTR SS:\[EBP-3DC0\]
  00EFAA99    8B02            MOV EAX,DWORD PTR DS:\[EDX\]
  00EFAA9B    50              PUSH EAX
  00EFAA9C    E8 2F7CFBFF     CALL 00EB26D0            <<---关键 F7 入!
  00EFAAA1    83C4 0C         ADD ESP,0C
  00EFAAA4    8D8D 40C1FFFF   LEA ECX,DWORD PTR SS:\[EBP-3EC0\]
  00EFAAAA    51              PUSH ECX
  

```

  
  跟进 CALL 00EB26D0  来到:  
代码:

```
  00EB26D0    55              PUSH EBP          <<---不管他.直接 retn.避开处理 IAT!
  00EB26D1    8BEC            MOV EBP,ESP
  00EB26D3    83EC 2C         SUB ESP,2C
  00EB26D6    833D 00A6F200 0>CMP DWORD PTR DS:\[F2A600\],0
  

```

  
  此时我们删除 VirtualProtect 断点.通过下 bp CreateThread 寻找真正的 OEP!  
  下好断点后.shift+f9!断下来后 Crtl+f9 回到 VM!来到这里:  
代码:

```
  00EE364C    50              PUSH EAX
  00EE364D    FF15 84E2F100   CALL DWORD PTR DS:\[F1E284\]               ; KERNEL32.CloseHandle
  00EE3653    5E              POP ESI
  00EE3654    5B              POP EBX
  00EE3655    8BE5            MOV ESP,EBP
  00EE3657    5D              POP EBP
  00EE3658    C3              RETN
  

```

  
  继续 Crtl+F9!!  
代码:

```
  00EFF1C5    B9 04A6F200     MOV ECX,0F2A604
  00EFF1CA    E8 B136FBFF     CALL 00EB2880
  00EFF1CF    E9 25010000     JMP 00EFF2F9
  00EFF1D4    E8 073CFDFF     CALL 00ED2DE0
  00EFF1D9    C705 3877F200 1>MOV DWORD PTR DS:\[F27738\],0F21C10
  00EFF1E3    E8 A894FDFF     CALL 00ED8690
  00EFF1E8    6A 00           PUSH 0
  00EFF1EA    E8 2143FEFF     CALL 00EE3510
  00EFF1EF    83C4 04         ADD ESP,4                 <<---出来后到这里!!
  00EFF1F2    B9 B8B4F200     MOV ECX,0F2B4B8
  00EFF1F7    E8 B4BFFBFF     CALL 00EBB1B0
  00EFF1FC    0FB6D0          MOVZX EDX,AL
  00EFF1FF    85D2            TEST EDX,EDX
  00EFF201    74 0C           JE SHORT 00EFF20F
  00EFF203    6A 01           PUSH 1
  00EFF205    B9 B8B4F200     MOV ECX,0F2B4B8
  00EFF20A    E8 11A6FCFF     CALL 00EC9820
  00EFF20F    C705 3877F200 0>MOV DWORD PTR DS:\[F27738\],0F21C08
  00EFF219    B9 04A6F200     MOV ECX,0F2A604
  00EFF21E    E8 5D36FBFF     CALL 00EB2880
  00EFF223    C745 F0 0000000>MOV DWORD PTR SS:\[EBP-10\],0
  00EFF22A    8D45 E8         LEA EAX,DWORD PTR SS:\[EBP-18\]
  00EFF22D    50              PUSH EAX
  00EFF22E    68 10F3EF00     PUSH 0EFF310
  00EFF233    FF15 74B1F200   CALL DWORD PTR DS:\[F2B174\]
  

```

  
  这样我们就到了 VM 中 Armadillo 运行程序的地方!  
  我们向下拉.寻找 call ecx.找到的第一个 call ecx 不是重点.我们需要的是第二个!!  
代码:

```
  00EFF29B    8B48 14         MOV ECX,DWORD PTR DS:\[EAX+14\]
  00EFF29E    51              PUSH ECX
  00EFF29F    8B55 08         MOV EDX,DWORD PTR SS:\[EBP+8\]
  00EFF2A2    8B42 10         MOV EAX,DWORD PTR DS:\[EDX+10\]
  00EFF2A5    50              PUSH EAX
  00EFF2A6    8B4D F4         MOV ECX,DWORD PTR SS:\[EBP-C\]
  00EFF2A9    2B4D E0         SUB ECX,DWORD PTR SS:\[EBP-20\]
  00EFF2AC    FFD1            CALL ECX                           <<---第一个 ECX.不重要.忽略!
  00EFF2AE    8945 FC         MOV DWORD PTR SS:\[EBP-4\],EAX
  00EFF2B1    EB 46           JMP SHORT 00EFF2F9
  00EFF2B3    8B55 08         MOV EDX,DWORD PTR SS:\[EBP+8\]
  00EFF2B6    833A 01         CMP DWORD PTR DS:\[EDX\],1
  00EFF2B9    75 3E           JNZ SHORT 00EFF2F9
  00EFF2BB    A1 04BBF200     MOV EAX,DWORD PTR DS:\[F2BB04\]
  00EFF2C0    8B0D 04BBF200   MOV ECX,DWORD PTR DS:\[F2BB04\]            ; UnpackMe.004CB378
  00EFF2C6    8B50 68         MOV EDX,DWORD PTR DS:\[EAX+68\]
  00EFF2C9    3351 34         XOR EDX,DWORD PTR DS:\[ECX+34\]
  00EFF2CC    A1 04BBF200     MOV EAX,DWORD PTR DS:\[F2BB04\]
  00EFF2D1    3350 60         XOR EDX,DWORD PTR DS:\[EAX+60\]
  00EFF2D4    8955 DC         MOV DWORD PTR SS:\[EBP-24\],EDX
  00EFF2D7    8B4D 08         MOV ECX,DWORD PTR SS:\[EBP+8\]
  00EFF2DA    8B51 04         MOV EDX,DWORD PTR DS:\[ECX+4\]
  00EFF2DD    52              PUSH EDX
  00EFF2DE    8B45 08         MOV EAX,DWORD PTR SS:\[EBP+8\]
  00EFF2E1    8B48 08         MOV ECX,DWORD PTR DS:\[EAX+8\]
  00EFF2E4    51              PUSH ECX
  00EFF2E5    6A 00           PUSH 0
  00EFF2E7    8B55 08         MOV EDX,DWORD PTR SS:\[EBP+8\]
  00EFF2EA    8B42 0C         MOV EAX,DWORD PTR DS:\[EDX+C\]
  00EFF2ED    50              PUSH EAX
  00EFF2EE    8B4D F4         MOV ECX,DWORD PTR SS:\[EBP-C\]
  00EFF2F1    2B4D DC         SUB ECX,DWORD PTR SS:\[EBP-24\]
  00EFF2F4    FFD1            CALL ECX                            <<----第二个 ECX.重要.跟进飞向 OEP!!
  00EFF2F6    8945 FC         MOV DWORD PTR SS:\[EBP-4\],EAX
  00EFF2F9    8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]
  00EFF2FC    5E              POP ESI
  00EFF2FD    8BE5            MOV ESP,EBP
  00EFF2FF    5D              POP EBP
  00EFF300    C3              RETN
  

```

  
  此时我们运行 LordPE.纠正一下映像大小.完整 dump 一下子进程.注意.PID 一定要为 2C8 的进程!!!  
  dump 好之后.运行 ImportREC.载入 PID 为 2C8 的进程!输入 OEP:5B52C.获取输入表.有 12 个无效指针.直接剪掉!然后修复程序.  
  修复后程序可运行啦~哈哈~  
  FastScan 查壳为 Borland Delphi 2006  
  PEID 查壳为 Borland Delphi 6.0  
  
  
\--------------------------------------------------------------------------------  
【经验总结】  
  菜鸟最近才开始学各大强壳手脱方法.  
  学习一个国外高人的脱壳方法笔记.没什么技术含量.拉出来与大家分享一下而已.  
  献丑了.  
  
\--------------------------------------------------------------------------------  
【版权声明】: 还不是踩在巨人的肩膀上说话?哪里来的版权.随便怎么样.哈哈~  
  
                                                       2009 年 07 月 02 日 15:27:41

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || UnpackMe\_Armadillo\_5.0.rar \[附件请到：[`bbs.pediy.com/showthread.php?t=92699`](http://bbs.pediy.com/showthread.php?t=92699) \]   
 |