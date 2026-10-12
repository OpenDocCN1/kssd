
# UltraEdit 9.10

> 原文：[`www.pediy.com/kssd/pediy06/pediy6127.htm`](https://www.pediy.com/kssd/pediy06/pediy6127.htm)

小弟学 `pj` 已经 2，3 个礼拜了，还没正式找过一款软件的注册码，这次就下定决心那 `ultraedit` v9.10 开刀。

以前刚学 `pj` 时爆破过它，去掉了时间限制和 `nag` 窗口，但是总觉得不爽，这次我参考了看雪精华四上 `oake` 写的文章，找到了他的注册码，在此谢谢 `oake` 前辈了。

废话讲完正式进入主题：

软件名称：`ultraedit` v9.10

软件简介：地球人都知道

下载地址：到处都是

破解目的：找出注册码

破解工具：`ollydbg` v1.09b

破解说明：该软件采用 `key` 文件保护方式，输入注册名：`qqj1228`，注册码：`12345-67890-abcde-fghij`（该形式是从 `oake` 前辈文章里看到的）后确定，要求重起，然后会在安装文件夹下生成与主文件同名的 `key` 文件以 `reg` 作为扩展名。重起后会读取该文件的内容判断注册与否。

于是用 `redfile` 下断，`f9` 运行来到最后一次中断处，看堆栈的内容可知是读 `key` 文件的，于是 `f8` 跟踪，可是我跟了半天就是没找到比较注册码的地方，只知道它把 `key` 文件里的内容读取后进行解密转换成输入的用户名和注册码，然后在哪比较楞是没找到。跟到后面会有一很大的循环反复调用 `peekmessaga`，我是新手不知道这个 `api` 是干什么的，就直接跳出循环。但是跳出之后软件就弹出注册窗口（已经过了试用期），这下咋办？

后又看了 `oake` 前辈的文章，想到可以 `CreateWindowExA` 下断，于是实行。

程序在 `redfile` `key` 文件的断点后中断于 `CreateWindowExA` 处，如下：

```asm
00474342   .  A1 48C55800         MOV     EAX, DWORD PTR DS:[58C548]
00474347   .  53                  PUSH    EBX                  ; /lParam = NULL
00474348   .  FF77 68             PUSH    DWORD PTR DS:[EDI+68]      ; |hInst = 00400000
0047434B   .  895D FC             MOV     DWORD PTR SS:[EBP-4], EBX   ; |
0047434E   .  53                  PUSH    EBX                  ; |hMenu = NULL
0047434F   .  FF70 1C             PUSH    DWORD PTR DS:[EAX+1C]      ; |hParent = 007D027C ('UltraEdit-32 - [Edit1]',class='Afx:400000:8:10011:0:4160323')
00474352   .  53                  PUSH    EBX                  ; |Height = 0
00474353   .  53                  PUSH    EBX                  ; |Width = 0
00474354   .  53                  PUSH    EBX                  ; |Y = 0
00474355   .  53                  PUSH    EBX                  ; |X = 0
00474356   .  53                  PUSH    EBX                  ; |style = WS_OVERLAPPED
00474357   .  56                  PUSH    ESI                  ; |WindowName = "UEAfx:200:0x0008:0:010"
00474358   .  56                  PUSH    ESI                  ; |Class = "UEAfx:200:0x0008:0:010"
00474359   .  53                  PUSH    EBX                  ; |Extstyle = 0
0047435A   .  FF15 ECF75300       CALL    DWORD PTR DS:[<&USER32.CreateWindowExA>
00474360   .  A3 74C25800         MOV     DWORD PTR DS:[58C274], EAX
00474365   .  8D87 00050000       LEA     EAX, DWORD PTR DS:[EDI+500]
0047436B   .  8858 10             MOV     BYTE PTR DS:[EAX+10], BL
0047436E   .  FF35 64275800       PUSH    DWORD PTR DS:[582764]
00474374   .  6A 01               PUSH    1
00474376   .  50                 PUSH    EAX
00474377   .  FF75 F0             PUSH    DWORD PTR SS:[EBP-10]
0047437A   .  E8 5A2BFAFF         CALL    UEDIT32.00416ED9           ; F8 过这个 CALL 后即可看到注册码
0047437F   .  83C4 10             ADD     ESP, 10
00474382   .  85C0               TEST    EAX, EAX
00474384   .  0F84 89010000       JE      UEDIT32.00474513
0047438A   .  391D 18C55800       CMP     DWORD PTR DS:[58C518], EBX
00474390   .  0F85 7D010000       JNZ     UEDIT32.00474513
00474396   .  68 74AE5700         PUSH    UEDIT32.0057AE74           ;  ASCII "Extension License"
0047439B   .  B9 68275800         MOV     ECX, UEDIT32.00582768      ;  ASCII "(??"
004743A0   .  E8 D8FB0800         CALL    UEDIT32.00503F7D
```

马上抄下注册码。`hehe` 不要高兴的太早，若用这个注册码的话刚进入程序时确实注册成功了，但是在你关闭程序时它又会把刚生成的 `key` 文件删了，又变成未注册的了。这又难倒我这个菜鸟了，`oake` 前辈的文章里也没提到，看来只能自力更生了。

既然它会删文件，那一定会调用 `deletefilea` 等 `api`，我就在此下断。

用 `ollydbg` 重新载入 `ultraedit`，`f9` 运行，返回程序关了它，马上被 `OD` 断下，向上找是否有指令可跳过它，来到：

```asm
004452B3   .  A1 80275800         MOV     EAX, DWORD PTR DS:[582780]
004452B8   .  FF35 64275800       PUSH    DWORD PTR DS:[582764]
004452BE   .  8945 E4             MOV     DWORD PTR SS:[EBP-1C], EAX
004452C1   .  E8 6AC40A00         CALL    UEDIT32.004F1730
004452C6   .  59                 POP     ECX                  ;  00E43E38
004452C7   .  83F8 0F             CMP     EAX, 0F              ; 比较注册码的位数是否小于 15，若小于 15 则跳走不删文件
004452CA   .  59                 POP     ECX                  ;  00E43E38
004452CB   .  0F82 F7000000       JB      UEDIT32.004453C8
004452D1   .  391D 14C55800       CMP     DWORD PTR DS:[58C514], EBX ; ebx＝0 为注册标志，与[58C514]＝0 比较，不等则跳走不删文件
004452D7   .  0F85 EB000000       JNZ     UEDIT32.004453C8
004452DD   .  391D 18C55800       CMP     DWORD PTR DS:[58C518], EBX ; 同上，与[58C518]＝1 比较，等则跳走不删文件。即若是不正确的注册码则不会删文件
004452E3   .  0F84 DF000000       JE      UEDIT32.004453C8
004452E9   .  0FB67D E4           MOVZX   EDI, BYTE PTR SS:[EBP-1C]  ; 取计算系数（d7）
004452ED   .  8BC7                MOV     EAX, EDI
004452EF   .  6A 0D               PUSH    0D
004452F1   .  99                  CDQ
004452F2   .  5B                 POP     EBX                  ;  00E43E38
004452F3   .  8B0D 64275800       MOV     ECX, DWORD PTR DS:[582764] ; 取注册码存放地址
004452F9   .  F7FB               IDIV    EBX
004452FB   .  0FBE41 10           MOVSX   EAX, BYTE PTR DS:[ECX+10] ; 取注册码的第 17 位（即第 3 段最后 1 位）的 ascii 值
004452FF   .  83C2 41             ADD     EDX, 41
00445302   .  3BC2                CMP     EAX, EDX             ; 取得的值与（d7%0d+41）的计算结果比较，不等则删文件
00445304   .  75 19               JNZ     SHORT UEDIT32.0044531F
00445306   .  8BC7                MOV     EAX, EDI
00445308   .  6A 18               PUSH    18
0044530A   .  99                  CDQ
0044530B   .  5F                 POP     EDI                  ;  00E43E38
0044530C   .  8A49 08             MOV     CL, BYTE PTR DS:[ECX+8]  ; 取注册码的第 9 位（即第 2 段第 3 位）的 ascii 值
0044530F   .  F7FF               IDIV    EDI
00445311   .  0FBEC1              MOVSX   EAX, CL
00445314   .  83C2 41             ADD     EDX, 41
00445317   .  3BC2                CMP     EAX, EDX             ; 取得的值与（d7%18+41）的计算结果比较，不等则删文件
00445319   .  0F84 A7000000       JE      UEDIT32.004453C6
0044531F   >  33DB                XOR     EBX, EBX
```

`hehe` 原来它在关闭时还要对注册码进行比较，不对就干掉你的 `key` 文件，总结以下：

注册名：`qqj1228`

注册码：`R2R0Z-A1X7E-02N2H-K503K`

这是小弟的处女破文，还望各位前辈高人批评指正。

`4dd466d0e1e0cc123afbe504da2462ca`
