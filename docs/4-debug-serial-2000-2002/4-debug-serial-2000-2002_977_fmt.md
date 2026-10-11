
# 易吉八字算命

> 原文：[`www.pediy.com/kssd/pediy04/kanxue121.htm`](https://www.pediy.com/kssd/pediy04/kanxue121.htm)

在第一篇 练了练 `SmartCheck`

这次用用 `TRW PW32DASM Winhex`

软件地址：`http://www.bazi-soft.com/`

使用工具：`PW32DASM` `Winhex`

首先用 `PW` 反汇编 然后查找“注册码错误”

寻找到以下地址 ----->

```
* Reference To: MFC42.Ordinal:0320, Ord:0320h
                          |
:00403E14 E847230100      Call 00416160
:00403E19 5E              pop esi
:00403E1A 84DB            test bl, bl
:00403E1C 5B              pop ebx
:00403E1D 7414            je 00403E33 <-----

<-- 改 JNE 就 OK

记下偏移地址（窗口底部）记下地址：@403E1D @3E1D

:00403E1F 8D4C2400        lea ecx, dword ptr [esp]
:00403E23 E8E8FD0000      call 00413C10
:00403E28 6A00            push 00000000
:00403E2A 6A00            push 00000000

* Possible StringData Ref from Data Obj ->"谢谢！注册成功"
                          |
:00403E2C 68DCE74100      push 0041E7DC
:00403E31 EB09            jmp 00403E3C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403E1D(C)
|
:00403E33 6A00            push 00000000
:00403E35 6A00            push 00000000

向上看:-----> 就是那个 JE

* Possible StringData Ref from Data Obj ->"注册码错误！"
                          |
:00403E37 68CCE74100      push 0041E7CC

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403E31(U)
```

打开 `Winhex` 并调入 易吉

找到 `@3E1D` --> `74148D4C`

`74` 改 `75` 存盘 退出 `Winhex`

再次运行 易吉 随便输入注册码

怎么样 ? 爆破成功了 !!!

使用工具：`TRW2000` `Winhex`

运行 `TRW` 易吉

选择帮助中的输入注册码

随便输入注册码 :`78787878`

按 `CTRL+N` 激活 `TRW`

`bpx hmemcpy`

`G` 继续执行程序 点" 确定 " 按钮

`TR` 窗口被激活自动弹出 `bc *` 取消中断

按几次 `F12` 来到易吉领空 `YJBZ！.TEX`

按 `F10` 跟踪到以下地址 ----->

```
017F:00403DE8 E8D9260100      CALL    MFC42!ord_000018BE
017F:00403DED 8D44240C        LEA     EAX,[ESP+0C]
017F:00403DF1 8D4C2408        LEA     ECX,[ESP+08]
017F:00403DF5 50              PUSH    EAX
017F:00403DF6 E8F5FF0000      CALL    00413DF0
017F:00403DFB 8B7664          MOV     ESI,[ESI+64]
017F:00403DFE 8B00            MOV     EAX,[EAX]
017F:00403E00 56              PUSH    ESI
017F:00403E01 50              PUSH    EAX <-----

<-- d eax 就能看见注册码 偶的是：2624524

G 出去 用注册码注册搞定

017F:00403E02 FF1548964100    CALL    MSVCRT!_mbscmp
017F:00403E08 83C408          ADD     ESP,BYTE +08
017F:00403E0B 8D4C240C        LEA     ECX,[ESP+0C]
017F:00403E0F 85C0            TEST    EAX,EAX
017F:00403E11 0F94C3          SETZ    BL
017F:00403E14 E847230100      CALL    MFC42!ord_00000320
017F:00403E19 5E              POP     ESI
017F:00403E1A 84DB            TEST    BL,BL
017F:00403E1C 5B              POP     EBX
017F:00403E1D 7414            JZ      00403E33 <-----

<--改 JNE 爆破成功

017F:00403E1F 8D4C2400        LEA     ECX,[ESP+00]
017F:00403E23 E8E8FD0000      CALL    00413C10
017F:00403E28 6A00            PUSH    BYTE +00
017F:00403E2A 6A00            PUSH    BYTE +00
017F:00403E2C 68DCE74100      PUSH    DWORD 0041E7DC
017F:00403E31 EB09            JMP     SHORT 00403E3C
017F:00403E33 6A00            PUSH    BYTE +00
017F:00403E35 6A00            PUSH    BYTE +00
017F:00403E37 68CCE74100      PUSH    DWORD 0041E7CC
017F:00403E3C E8B3250100      CALL    MFC42!ord_000004B0
017F:00403E41 8D4C2400        LEA     ECX,[ESP+00]
017F:00403E45 C7442410FFFFFFFF MOV     DWORD [ESP+10],FFFFFFFF
017F:00403E4D E8AEFD0000      CALL    00413C00
017F:00403E52 8B4C2408        MOV     ECX,[ESP+08]
017F:00403E56 64890D00000000  MOV     [FS:00],ECX
017F:00403E5D 83C414          ADD     ESP,BYTE +14
```

`4dd466d0e1e0cc123afbe504da2462ca`
