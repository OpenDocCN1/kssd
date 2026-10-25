
# 媒体 2000V6.0-高级复唱机

> 原文：[`www.pediy.com/kssd/pediy03/forum656.htm`](https://www.pediy.com/kssd/pediy03/forum656.htm)

简单破解：

1. 修改注册表：用 `RegSnap` 比较运行 `mt2000.exe` 前后，发现注册表中如下键值发生变化，下次运行时将注册表的值改过来即可。
   ```
   HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\VideoOutType\
   VideoOutVolue=0x00000024(36) <--还能运行 9 次（键值不能小于该值）
   VideoOutVolue=ox00000024(36) （两个键值都改）
   ```
2. 修改 `mt2000.exe`，使之可运行次数不减少：
   先用 `W32dasm` 反编译 `mt2000.exe`，在"裁判"->"串式数据参考"中查找"您未能成功输入本版注册号，只能在一定期限内..."字样，双击之，来到反汇编代码处，向上走几步，看到 `00410C09 2BF0 sub esi,eax`，这就是减少使用次数的代码。用 `HIEW` 将其改为 `9090 nop nop` 即可。

高级破解：求助！！！！！

3. 修改 `mt2000.exe`，使任意输入注册号都能注册成功。（用 `TRW2000` 跟踪半天，没能找到关键地方。）
4. 跟踪到进行注册号比较的地方，直接找到注册号（也没成功！5555555）

### 一、流程分析

先运行 `TRW2000 1.23` 注册版，再运行媒体 2000，随便乱输注册码，确定，跳出窗口提示注册码错，按 `ctrl-N`，下 `pmodule` 命令，返回出错窗口，点确定，立刻被拦，按 2 次 `F10` 来到 `40f41d` 处，往上看：

```
0167:0040F0D1  LEA      ECX,[ESP+48]
0167:0040F0D5  MOV      BYTE [ESP+052C],0A
0167:0040F0DD  CALL    004593E8　　　　　　　　显示输入框
0167:0040F0E2  CMP      EAX,BYTE +01　　　　　　按确定？
0167:0040F0E5  JNZ      NEAR 0040F40E　　　　　　否
0167:0040F0EB  LEA      EAX,[ESP+A4]
0167:0040F0F2  LEA      ECX,[ESP+14]
0167:0040F0F6  PUSH    EAX
0167:0040F0F7  CALL    0045DD86
0167:0040F0FC  MOV      ECX,[ESP+14]
0167:0040F100  PUSH    DWORD 004A7B48
0167:0040F105  PUSH    ECX
0167:0040F106  CALL    0043319B
0167:0040F10B  ADD      ESP,BYTE +08
0167:0040F10E  TEST    EAX,EAX　　　　　　　　　注册码为空？
0167:0040F110  JZ      0040F15C　　　　　　　　　是
0167:0040F112  MOV      EDX,[ESP+14]
0167:0040F116  MOV      EAX,[EDX-08]　　　　　　　取注册码长度
0167:0040F119  CMP      EAX,BYTE +0C
0167:0040F11C  JL      0040F15C　　　　　　　　小于 12 则错
0167:0040F11E  PUSH    BYTE +04
0167:0040F120  LEA      EAX,[ESP+14]
0167:0040F124  PUSH    BYTE +08
0167:0040F126  PUSH    EAX
0167:0040F127  LEA      ECX,[ESP+20]
0167:0040F12B  CALL    0044F1DB　　　　　　　　取注册码的第 8－12 位
0167:0040F130  MOV      EAX,[EAX]
0167:0040F132  PUSH    EAX
0167:0040F133  CALL    0043362C　　　　　　　　变成数值
0167:0040F138  ADD      ESP,BYTE +04
0167:0040F13B  LEA      ECX,[ESP+10]
0167:0040F13F  MOV      ESI,EAX　　　　　　　　存入 ESI
0167:0040F141  CALL    0045DC4D
0167:0040F146  CMP      ESI,BYTE +64
0167:0040F149  JNL      0040F158　　　　　　　　小于 64 则错
0167:0040F14B  PUSH    BYTE +00
0167:0040F14D  PUSH    EBP
0167:0040F14E  PUSH    DWORD 004A1C9C
0167:0040F153  JMP      0040F416
0167:0040F158  CMP      ESI,EDI　　　　　　　　等于识别码后 4 位？
0167:0040F15A  JZ      0040F169　　　　　　　　等于
0167:0040F15C  PUSH    BYTE +00
0167:0040F15E  PUSH    EBP
0167:0040F15F  PUSH    DWORD 004A1C9C
0167:0040F164  JMP      0040F416　　　　　　　　出错
0167:0040F169  LEA      ECX,[EDI+EDI*4]　　　　以下校验注册码的真假
0167:0040F16C  LEA      EDX,[EDI+EDI*8]
0167:0040F16F  LEA      EAX,[EDI+ECX*8]
0167:0040F172  LEA      ESI,[EDI+EDX*2]
0167:0040F175  LEA      ECX,[EAX+EAX*2]
.
.
.
0167:0040F300  LEA      EAX,[ECX+EAX*4]
0167:0040F303  LEA      EAX,[EAX+EAX*4]
0167:0040F306  LEA      ECX,[EAX+EAX*4]
0167:0040F309  LEA      EAX,[ESI+ECX*4]
0167:0040F30C  LEA      EAX,[EAX+EAX*4]
0167:0040F30F  LEA      EAX,[EAX+EAX*4]
0167:0040F312  LEA      EDX,[EDX+EAX*4-1C]
0167:0040F316  MOV      EAX,[ESP+44]
0167:0040F31A  CMP      EAX,EDX　　　　　　　　注册码为真？
0167:0040F31C  JNZ      NEAR 0040F3C1　　　　　否，出错
```

弄清以上流程，爆破就容易了。

### 二、注册码的校验算法

1. 将输入的 12 位注册码分成 2 部分：前 8 位设为 `X`，后 4 位设为 `Y`（均为 10 进制，以下同）；
2. 将 `Y*123` 的前 3 位与 `Y*456` 的第 2、3、4 位串起来成为一个 6 位数，设为 `Z`；
3. 计算 `X-Z`，设为 `M`；
4. 将 `M` 从高位到低位分成 4 个 2 位数，设为 `A`、`B`、`C`、`D`；
5. 将 `A`、`B`、`C`、`D` 重新组合成新的 8 位数 `CBDA`，计算 `CBDA-15111028`，设为 `N`；
6. 比较 `N` 与识别码的前 8 位，相等则输入的注册码为真，否则为假。

### 三、注册码的生成算法

1. 将软件给出的 12 位识别码分成 2 部分：前 8 位设为 `X`，后 4 位设为 `Y`（均为 10 进制，以下同）；
2. 将 `Y*123` 的前 3 位与 `Y*456` 的第 2、3、4 位串起来成为一个 6 位数，设为 `Z`；
3. 计算 `X+15111028`，设为 `M`；
4. 将 `M` 从高位到低位分成 4 个 2 位数，设为 `A`、`B`、`C`、`D`；
5. 将 `A`、`B`、`C`、`D` 重新组合成新的 8 位数 `DBAC`，计算 `DBAC+Z`，设为 `N`；
6. 将 `N` 与识别码的后 4 位串起来，形成注册码。

### 四、注册成功标志

注册成功标志放在注册表的 `HKEY_USERS\.Default\RemoteAccess\Identf\Licn` 处。

### 附录：《媒体 2000V6.0》算号器

请用 `VB5` 以上编译。

```vb
'《媒体 2000V6.0》算号器，请用 VB5 以上编译
'说明：Text1 为机器识别号（12 位数字），Text2 为注册码（自动算出）
'作者：小明　2001.2.20

Private Sub Text1_Change()
    s = Trim(Text1.Text)
    If Len(s) = 12 Then
        x = Val(Left(s, 8))
        y = Val(Right(s, 4))
        z = Val(Left(Trim(Str(y * 123)), 3) + Mid(Trim(Str(y * 456)), 2, 3))
        
        x = x + 28
        a = x Mod 100
        x = x \ 100 + 10
        
        d = x Mod 100
        x = x \ 100 + 11
        
        b = x Mod 100
        x = x \ 100 + 15
        
        c = x Mod 100
        
        z = a * 1000000 + b * 10000 + c * 100 + d + z
        
        Text2.Text = Trim(Str(z)) + Right(s, 4)
    Else
        Text2.Text = ""
    End If
End Sub
```
