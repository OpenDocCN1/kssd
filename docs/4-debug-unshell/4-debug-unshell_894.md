

# PEQueKe0.06 花指令壳学习笔记【这壳太猛了】

> 原文：[`www.pediy.com/kssd/pediy12/135549.html`](https://www.pediy.com/kssd/pediy12/135549.html)

这壳好像是一猛壳！  
这个壳成功的运用了花指令生成引擎，小弟最终还是没能分析出 OEP 出来，我就给大家看下学习记录算了。  
我分析到得保护技术：花指令，内存访问异常，花指令用的好成熟，好像还有调用堆栈执行代码异常终止。  
花指令在于实践，所以大家一定要拿着 OD 弄一下，  
程序以一个 CALL 004050AA 开始，然后就进行自定位，  

代码:

```
004050AA    5D              POP EBP             CALL 会压入下条指令的地址保存，以便返回
004050AB    81ED 05000000   SUB EBP,5           定位到程序代码的开始处，以便控制外壳段

接下来就是加载外壳段储存的 Kernel32.dll，然后装载，获得 VirtualAlloc 的地址，再调用 VirtualAlloc 分配指定大小的内存，用来接收解码后的外壳段代码
004050B1    8D75 3D         LEA ESI,DWORD PTR SS:\[EBP+3D\]   ; "Kernel32.dll"
004050B4    56              PUSH ESI                        ; Kernel32.dll 的偏移地址
004050B5    FF55 31         CALL DWORD PTR SS:\[EBP+31\]      ; GetModuleHandleA，获得模块基址
004050B8    8DB5 81000000   LEA ESI,DWORD PTR SS:\[EBP+81\]   ; “VirtualAlloc”
004050BE    56              PUSH ESI                        ; VirtualAlloc 的偏移地址
004050BF    50              PUSH EAX                        ; Kernel32.dll 的模块基址
004050C0    FF55 2D         CALL DWORD PTR SS:\[EBP+2D\]      ; GetProcAddress 获得函数地址
004050C3    8985 8E000000   MOV DWORD PTR SS:\[EBP+8E\],EAX   ; VirtualAlloc 函数地址保存的地方
004050C9    6A 04           PUSH 4                          ; 保护方式 PAGE\_READWRITE
004050CB    68 00100000     PUSH 1000                       ; 分配的类型 MEM\_COMMIT
004050D0    68 28900000     PUSH 9028                       ; 分配的大小
004050D5    6A 00           PUSH 0                          ; 分配大小的起始位置，用来计算
004050D7    FF95 8E000000   CALL DWORD PTR SS:\[EBP+8E\]      ; CALL VirtualAlloc

分配好虚拟内存后，然后解密外壳段代码拷贝到分配的内存里面，这个过程中随机生成花指令，然后在 JMP 跳到分配的内存里面去，执行代码
004050DD    50              PUSH EAX                        ; 虚拟内存的地址
004050DE    8B9D 7D000000   MOV EBX,DWORD PTR SS:\[EBP+7D\]   ; 要解码的偏移，19C
004050E4    03DD            ADD EBX,EBP                     ; 加上程序开始的地址 00405000 就是 0040519C 这个地方了
004050E6    50              PUSH EAX                        ; 分配内存的虚拟地址，参数 2
004050E7    53              PUSH EBX                        ; 原来的位置，参数一
004050E8    E8 04000000     CALL PECompac.004050F1          ; CALL 解码拷贝
004050ED    5A              POP EDX                         ; 恢复虚拟地址的地址，也就是 MOV 前面那个 push eax
004050EE    55              PUSH EBP                        ; 外壳代码基址
004050EF    FFE2            JMP EDX                         ; JMP 虚拟内存
现在开始解密后的内容，这段代码好多花指令，大家要注意：
003D0000   /EB 01           JMP SHORT 003D0003    ; 跳到 3D0003
003D0002   |90              NOP                   ; 这里的 OF 生成的花指令，原指令是 RDTSC，我们右键二进制编辑把 0x0F NOP 掉，保留 0x31 操作码
003D0003    31F0            XOR EAX,ESI           ; 分配内存大小和储存字符串 VirtualAlloc 的地址异或
003D0005    EB 0C           JMP SHORT 003D0013    ;跳到下面 PUSH ECX 那里
003D0007    33C8            XOR ECX,EAX           ;ECX 和 EAX 异或
003D0009    EB 03           JMP SHORT 003D000E    ;跳到花指令，我现在把花指令操作码 0x0F NOP 掉了，
003D000B    EB 09           JMP SHORT 003D0016     跳到下面 XOR EAX,ESI 那里，那里原来也是花指令
003D000D    90              NOP                    原来的指令是，003D000D  0F 59 74 05 MULPS XMM6,DQWORD PTR SS:\[EBP+EAX+75\]，把操作码 0x0F 改为 90
003D000E    59              POP ECX                恢复 ECX 的值
003D000F    74 05           JE SHORT 003D0016      ZF 等于 1 就跳
003D0011  ^ 75 F8           JNZ SHORT 003D000B     ZF 等于 0 就向上跳
003D0013    51              PUSH ECX
003D0014  ^ EB F1           JMP SHORT 003D0007     ;跳到 XOR ECX，EAX 那里
003D0016    EB 01           JMP SHORT 003D0019
003D0018    90              NOP                     同上，0x0F 改为 90
003D0019    31F0            XOR EAX,ESI             EAX 和 ESI 异或
003D001B    EB 0C           JMP SHORT 003D0029      跳到下面 PUSH ECX 那里
003D001D    33C8            XOR ECX,EAX             异或
003D001F    EB 03           JMP SHORT 003D0024      跳到花指令，我们把 0x0F 改为 90
003D0021    EB 09           JMP SHORT 003D002C      跳到 XOR ESI,ESI 那里 ESI，清零
003D0023    90              NOP
003D0024    59              POP ECX                 恢复 ECX 的值
003D0025    74 05           JE SHORT 003D002C       ZF 等于 1 就跳
003D0027  ^ 75 F8           JNZ SHORT 003D0021      ZF 是 0 就向上跳转，跳到 NOP 上面那里
003D0029    51              PUSH ECX                保存 ECX 的值
003D002A  ^ EB F1           JMP SHORT 003D001D      跳到上面 XOR ECX,EAX 那里
003D002C    33F6            XOR ESI,ESI             ESI 清零
003D002E    E8 10000000     CALL 003D0043

现在开始学习大量的花指令了：
指令：
003D002C    33F6            XOR ESI,ESI
003D002E    E8 10000000     CALL 003D0043
{
    003D0041    C783 64FF3500 0>MOV DWORD PTR DS:\[EBX+35FF64\],64000000
    003D004B    8925 00000000   MOV DWORD PTR DS:\[0\],ESP
    003D0051    AD              LODS DWORD PTR DS:\[ESI\]
}
修改后的代码：
003D0041    90              NOP
003D0042    90              NOP
003D0043    64:FF35 0000000>PUSH DWORD PTR FS:\[0\]
003D004A    64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP
003D0051    AD              LODS DWORD PTR DS:\[ESI\]
003D0052    CD 20           INT 20         反跟踪代码
指令：
003D0054    68 9F6F56B6     PUSH B6566F9F
003D0059    50              PUSH EAX
003D005A    E8 5D000000     CALL 003D00BC
{
    003D00BC    58              POP EAX
    003D00BD    EB FF           JMP SHORT 003D00BE
}
修改后：
003D00BC    58              POP EAX
003D00BD    90              NOP
003D00BE    FFF0            PUSH EAX
003D00C0    EB FF           JMP SHORT 003D00C1
指令：
003D00C0   /EB FF           JMP SHORT 003D00C1
003D00C2    C083 E8FDEBFF 3>ROL BYTE PTR DS:\[EBX+FFEBFDE8\],30            ; 移位常量超出 1..31 的范围
003D00C9    E8 C9000000     CALL 003D0197                             //受到前面指令的影响了
修改后：
003D00C0    90              NOP
003D00C1    FFC0            INC EAX
003D00C3    83E8 FD         SUB EAX,-3
003D00C6    EB FF           JMP SHORT 003D00C7                    大家看出来没，双花指令
003D00C8    30E8            XOR AL,CH
修改后：
003D00C6    90              NOP
003D00C7    FF30            PUSH DWORD PTR DS:\[EAX\]
代码
003D00C7    FF30            PUSH DWORD PTR DS:\[EAX\]
003D00C9    E8 C9000000     CALL 003D0197
{
    003D0196  ^\\EB 83           JMP SHORT 003D011B                           花指令
    003D0198    C017 EB         RCL BYTE PTR DS:\[EDI\],0EB                    ; 移位常量超出 1..31 的范
}
003D0196    90              NOP
003D0197    83C0 17         ADD EAX,17       修改后
花指令：
003D019A   /EB FF           JMP SHORT 003D019B          花
003D019C  ^ 70 ED           JO SHORT 003D018B
修改后：
003D019A    90              NOP
003D019B    FF70 ED         PUSH DWORD PTR DS:\[EAX-13\]
这个地方这里比较厉害，咋改都不对：
003D0081   /75 09           JNZ SHORT 003D008C
003D0083   |81F3 EBFF52BA   XOR EBX,BA52FFEB
003D0089   |0100            ADD DWORD PTR DS:\[EAX\],EAX
003D008B   |83EB FC         SUB EBX,-4
003D008E    4A              DEC EDX
003D008F    FF71 0F         PUSH DWORD PTR DS:\[ECX+F\]
哪位高人解释一下

```

后面这样的代码还很多，现在总结一下心得：  
当我们遇到的花指令比较多的时候，一定要按代码执行流程修改，不然看见一个就修改一个得话，会弄乱代码的

上传的附件
| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) || PEQueKe 学习记录.zip  
 |