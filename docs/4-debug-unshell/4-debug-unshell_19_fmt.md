
# pklite32、Shrinker 3.4 和 NeoLite

> 原文：[`www.pediy.com/kssd/pediy02/forum183.htm`](https://www.pediy.com/kssd/pediy02/forum183.htm)

## 脱壳----对用 pklite32 加壳的程序进行手动脱壳

- **目标文件**：`pklite32w.exe`
- **加壳方式**：`pklite32`
- **所用工具**：`trw2000 v1.22`
- **作者**：iis / 属于梦醒时分 [http://hacking.wofly.com]

1. 用 `trw2000` 的 `Loader` 加载加壳程序。
2. 按一下 `f10`, 将停在下面的地方：

```
0167:00607005  PUSH    DWORD 00627F84    《——停在这里,一直按 f10
0167:0060700A  PUSH    DWORD 00
0167:0060700F  CALL    00627F84
0167:00607014  JMP     004117B0    《——执行完这个指令，执行命令 makepe 文件名
0167:00607019  INC     EAX
0167:0060701A  SUB     [EBX],AH
```

3. 脱壳成功。

## 脱壳----对用 Shrinker 3.4 加壳的程序进行手动脱壳

- **加壳方式**：`Shrinker 3.4`
- **所用工具**：`trw2000 v1.22`
- **作者**：iis / 属于梦醒时分 [http://hacking.wofly.com]

1. 用 `trw2000` 的 `Loader` 加载加壳程序。
2. 按一下 `f10`, 将停在下面的地方：

```
0167:004365AF  PUSH    EBP    《——停在这里
0167:004365B0  MOV     EBP,ESP
0167:004365B2  PUSH    ESI
0167:004365B3  PUSH    EDI
0167:004365B4  JNZ     00436621
0167:004365B6  PUSH    DWORD 0100
0167:004365BB  CALL    004370D1
```

3. 一直按 `f10`:

```
0167:00436619  CALL    `KERNEL32!GetModuleFileNameA`
0167:0043661F  JMP     SHORT 00436624
0167:00436621  MOV     ESI,[EBP+08]
0167:00436624  CALL    00435000
0167:00436629  PUSH    DWORD [EBP+10]
0167:0043662C  PUSH    DWORD [EBP+0C]
0167:0043662F  PUSH    ESI
0167:00436630  CALL    0043663B      《——按 f8 进入
0167:00436635  POP     EDI
0167:00436636  POP     ESI
0167:00436637  POP     EBP
0167:00436638  RET     0C
```

4. 再按 `f10` 到达下面的地方：

```
0167:0043667C  INC     DWORD [004311C0]
0167:00436682  CALL    00435CB2    《——此处将弹出一个消息框，按‘是‘将返回 tr w2000.
0167:00436687  MOV     ESI,[004311C0]
0167:0043668D  TEST    ESI,ESI
0167:0043668F  JZ      004366FF
0167:00436691  CMP     DWORD [004311C4],BYTE +00
0167:00436698  JNZ     004366C0
0167:0043669A  CMP     DWORD [00433774],BYTE +00
0167:004366A1  JZ      004366C0
0167:004366A3  MOV     EAX,[00433774]
0167:004366A8  ADD     EAX,[00433898]
0167:004366AE  MOV     [EBP-20],EAX
0167:004366B1  PUSH    DWORD [EBP+10]
0167:004366B4  PUSH    DWORD [EBP+0C]
0167:004366B7  PUSH    DWORD [EBP+08]
0167:004366BA  CALL    NEAR [EBP-20] 《——按 f8 进入后，执行命令 makepe 文件名
0167:004366BD  MOV     [EBP-1C],EAX
```

5. 脱壳成功。

## 脱壳----对用 NeoLite 加壳的程序进行手动脱壳

- **目标文件**：`NeoLite（2.0）.exe`
- **加壳方式**：`NeoLite`
- **所用工具**：`trw2000 v1.22`
- **作者**：iis / 属于梦醒时分 [http://hacking.wofly.com]

1. 用 `trw2000` 的 `Loader` 加载加壳程序。
2. 将停在下面的地方：

```
0167:0041C1A8  JMP     0041C253 《——停在这里
0167:0041C1AD  MOV     EAX,A80041DA
0167:0041C1B2  ROL     BYTE [ECX+00],AC
0167:0041C1B6  ROL     BYTE [ECX+00],00
0167:0041C1BA  ADD     [EAX],AL
0167:0041C1BC  ADD     [EAX+6A00004A],BH
0167:0041C1C2  RET     41
```

3. 一直按 `f10`:

```
0167:0041C262  INC     BYTE [0041C252]
0167:0041C268  JMP     EAX      《——跳到真正的入口点
0167:0041C26A  CMP     BYTE [0041C252],00
0167:0041C271  JNZ     0041C286
```

4. 跳到下面地方：

```
0167:004073F3  PUSH    EBP        《——跳到这，执行命令 makepe 文件名
0167:004073F4  MOV     EBP,ESP
0167:004073F6  PUSH    BYTE -01
0167:004073F8  PUSH    DWORD 0040D1A0
0167:004073FD  PUSH    DWORD 00409A3C
0167:00407402  MOV     EAX,[FS:00]
0167:00407408  PUSH    EAX
```

5. 脱壳成功。

***
4dd466d0e1e0cc123afbe504da2462ca
