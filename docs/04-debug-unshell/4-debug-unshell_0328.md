
# Win2K 下寻找 ASProtect OE

> 原文：[`www.pediy.com/kssd/pediy04/kanxue038.htm`](https://www.pediy.com/kssd/pediy04/kanxue038.htm)

正好在 `Fravia` 论坛上看到有人问这个问题，试着跟了一下 `Tag & Rename v1.96`（主程序 751,616 字节），发现其 `OEP` 为 `5569EC`，和 `zombieys` 得到的 `556a00` 不同。

方法如下：

1. 利用 `ASProtect` 的 API 重定向/模拟特性。对于 `Tag & Rename` 来说，下面几个 API 被模拟，利用下面的几个 API 中的某一个作为断点（不要用使用频率太高的，比如不要用被频繁调用的 `GetModuleHandleA` 作为断点），比如用 `bpx GetVersion`，两次中断之后就到了：

```
001B:00F7C7BA  8BC0              MOV       EAX,EAX
001B:00F7C7BC  6A00              PUSH      00
001B:00F7C7BE  E8897CFFFF        CALL      KERNEL32!GetModuleHandleA
001B:00F7C7C3  A34C36F800        MOV       [00F8364C],EAX
001B:00F7C7C8  E8977CFFFF        CALL      KERNEL32!GetVersion
001B:00F7C7CD  A34436F800        MOV       [00F83644],EAX
001B:00F7C7D2  68AC35F800        PUSH      00F835AC
001B:00F7C7D7  E8907CFFFF        CALL      KERNEL32!GetVersionExA
001B:00F7C7DC  E83B7CFFFF        CALL      KERNEL32!GetCurrentProcess
001B:00F7C7E1  A34836F800        MOV       [00F83648],EAX
001B:00F7C7E6  E8397CFFFF        CALL      KERNEL32!GetCurrentProcessId
001B:00F7C7EB  A35036F800        MOV       [00F83650],EAX
001B:00F7C7F0  E81F7CFFFF        CALL      KERNEL32!GetCommandLineA
001B:00F7C7F5  A34036F800        MOV       [00F83640],EAX
001B:00F7C7FA  C3                RET
```

2. 执行完上面的 `ret` 指令之后，紧跟着的就是 20 多个精心构造的 `SEH` 结构。每个 `SEH` 结构大致如下。由于 `EAX` 为 0，所以 `xor` 指令会引发异常。

.......................................

```
001B:00F800EA  648920            MOV       FS:[EAX],ESP  //准备 ERR 结构
001B:00F800ED  3100              XOR       [EAX],EAX
001B:00F800EF  EB01              JMP       00F800F2
```

3. 按 `F8` 单步执行上面的 `xor` 指令，会进入 `KiUserExceptionDispatcher`：

`ntdll!KiUserExceptionDispatcher`

```
001B:77F9FB8C  8B4C2404          MOV       ECX,[ESP+04]
001B:77F9FB90  8B1C24            MOV       EBX,[ESP]
001B:77F9FB93  51                PUSH      ECX
001B:77F9FB94  53                PUSH      EBX
001B:77F9FB95  E886B3FeFF        CALL      77F8AF20
001B:77F9FB9A  0AC0              OR        AL,AL
001B:77F9FB9C  740C              JZ        77F9FBAA
001B:77F9FB9E  5B                POP       EBX
001B:77F9FB9F  59                POP       ECX
001B:77F9FBA0  6A00              PUSH      00
001B:77F9FBA2  51                PUSH      ECX
001B:77F9FBA3  E8EE2BFFFF        CALL      ntdll!NtContinue
001B:77F9FBA8  EB0B              JMP       77F9FBB5
001B:77F9FBAA  5B                POP       EBX
001B:77F9FBAB  59                POP       ECX
001B:77F9FBAC  6A00              PUSH      00
001B:77F9FBAE  51                PUSH      ECX
001B:77F9FBAF  53                PUSH      EBX
001B:77F9FBB0  E83D65FEFF        CALL      ntdll!NtRaiseException
```

4. 继续进入 `NtContinue`，会调用系统服务：

`ntdll!NtContinue`

```
001B:77F92796  B81C000000        MOV       EAX,0000001C
001B:77F9279B  8D542404          LEA       EDX,[ESP+04]
001B:77F9279F  CD2E              INT       2E
001B:77F927A1  E9467A0000        JMP       77F9A1EC
```

5. 按 `F8` 执行上面的 `INT` 指令，进入系统服务分发程序，最终进入下面的 `call`：

```
0008:8053167C  C1E902            SHR       ECX,02
0008:8053167F  8BFC              MOV       EDI,ESP
0008:80531681  3B35E89E5380      CMP       ESI,[ntoskrnl!MmUserProbeAddress]
0008:80531687  0F83E4010000      JAE       80531871
0008:8053168D  F3A5              REPZ MOVSD
0008:8053168F  FFD3              CALL      EBX
```

6. 进去之后一直按 `F10`，看见系统调用会用 `iretd` 指令返回：

```
0008:8053191A  66837D6C08        CMP       WORD PTR [EBP+6C],08
0008:8053191F  740C              JZ        8053192D
0008:80531921  8D6530            LEA       ESP,[EBP+30]
0008:80531924  0FA9              POP       GS
0008:80531926  07                POP       ES
0008:80531927  1F                POP       DS
0008:80531928  8D6550            LEA       ESP,[EBP+50]
0008:8053192B  0FA1              POP       FS
0008:8053192D  8D6554            LEA       ESP,[EBP+54]
0008:80531930  5F                POP       EDI
0008:80531931  5E                POP       ESI
0008:80531932  5B                POP       EBX
0008:80531933  5D                POP       EBP
0008:80531934  66817C24088000   CMP       WORD PTR [ESP+08],0080
0008:8053193B  7749              JA        80531986
0008:8053193D  83C404            ADD       ESP,04
0008:80531940  CF                IRETD
```

7. 清除所有断点，在上面的 `iretd` 处设个断点（也可以在 `int 2E` 上设断点，总之是以不变应万变，即不管什么类型的应用程序异常都会在这里集中处理。`iretd` 处可能是全部系统调用的出口，不仅仅包括 `NtContinue` 系统调用，还有其它的系统调用也经过这里）。之后按 `F5` 执行，由于有 26 个左右的 `SEH` 结构，所以会在上面的 `iretd` 处中断近 26 次。

8. 26 次之后在上述断点处改按 `F8`，会继续执行 `ASProtect` 加的壳。此后再经过几次 `ret` 之后，紧跟着的基本上是几个大的循环。每个循环的标志大致如下。只要在循环的出口处（即 `JNZ` 下面的那条指令处）设个断点就可以把循环跳过。

......................

```
001B:00F93608  0FBFF1            MOVSX     ESI,CX
001B:00F9360B  83EA01            SUB       EDX,01
001B:00F9360E  80C3C3            ADD       BL,C3
001B:00F93611  4A                DEC       EDX
001B:00F93612  4A                DEC       EDX
001B:00F93613  4A                DEC       EDX
001B:00F93614  51                PUSH      ECX
001B:00F93615  B374              MOV       BL,74
001B:00F93617  5E                POP       ESI
001B:00F93618  81FAF0FFFFFF      CMP       EDX,FFFFFFF0
001B:00F9361E  0F851D000000      JNZ       00F93641    //循环
001B:00F93624  66BB6E0A          MOV       BX,0A6E
001B:00F93628  E929000000        JMP       00F93656
```

9. 最终到达入口处：

```
001B:00F93655  855B58            TEST      [EBX+58],EBX
001B:00F93658  0524BCDA0E        ADD       EAX,0EDABC24
001B:00F9365D  5C                POP       ESP
001B:00F9365E  03C3              ADD       EAX,EBX
001B:00F93660  8944241C          MOV       [ESP+1C],EAX
001B:00F93664  61                POPAD
001B:00F93665  FFE0              JMP       EAX         //跳到 OEP 去
```

最通用的是 `CreateFileA`，因为 `ASProtect` 的壳一般都加上了自校验功能，所以要用 `CreateFileA` 去打开自己的 `exe`，故可以用这个做断点。之后再经过 16 次左右的 `IRETD` 指令，就快到入口了。

附：

`ReGet Deluxe 2.0 (build 93)`

`OEP = 004A38DE`

`http://download.reget.com/dl/jr/regetdx_093.exe` `4dd466d0e1e0cc123afbe504da2462ca`
