
# DISKdata v3.2.0

> 原文：[`www.pediy.com/kssd/pediy03/forum358.htm`](https://www.pediy.com/kssd/pediy03/forum358.htm)

## DISKdata v3.2.0 暴力破解

对不起大家了，先前发的那个贴子太疏忽了，没有验证便发了上去，我今天试了一下结果不能用，%$&\*$%%#·，结果贻笑大方，结果我的脸象个 XX。

下面我把正确的破解方法写出来。

**工具**：`TRW2K`，`UltraEdit`

**步骤**：

1. 用 `TRW2K` 载入 `DISKdata`，按 `F10`，一直到出现 `NAG` 窗。
2. 退出程序，被 `TRW` 拦截，看一下此时的 `eip`，在其上的 `Call` 上设断。
   ```
   :0049101F E838D6FBFF      call 0044E65C   **在此设断**
   :00491024 A1643A4900      mov eax, dword ptr [00493A64]  **TRW 将在此处中断**
   :00491029 8B00            mov eax, dword ptr [eax]
   :0049102B E8ACD6FBFF      call 0044E6DC
   :00491030 E8B32AF7FF      call 00403AE8
   ```
3. 再次载入程序，中断后 `F8` 进入，再一直按 `F10`，直到 `NAG` 窗出现。
4. 当 `NAG` 窗退出时，`TRW` 中断，按 `F4` 看看用户屏幕有什么变化，没有。此时 `TRW` 屏幕显示如下：
   ```
   :0048AB6C 8B0DF8394900      mov ecx, dword ptr [004939F8]
   :0048AB72 A1643A4900        mov eax, dword ptr [00493A64]
   :0048AB77 8B00              mov eax, dword ptr [eax]
   :0048AB79 8B15644C4800      mov edx, dword ptr [00484C64]
   :0048AB7F E8D83AFCFF        call 0044E65C
   :0048AB84 A1F8394900        mov eax, dword ptr [004939F8]  **在此设断**
   :0048AB89 8B00              mov eax, dword ptr [eax]
   :0048AB8B 8B10              mov edx, dword ptr [eax]
   :0048AB8D FF92D8000000      call dword ptr [edx+000000D8]  **出现 NAG 窗**
   :0048AB93 A1F8394900        mov eax, dword ptr [004939F8]  **TRW 将在此中断**
   :0048AB98 8B00              mov eax, dword ptr [eax]
   :0048AB9A E8B983F7FF        call 00402F58
   ```
   可以看出 `48ab8d` 是调用 `NAG` 窗的模块，有兴趣的同志可以进去看看，里面便是判断是否注册的地方。
5. 把断点设于 `48AB84`，重新 `LOAD`，中断后，`R eip 48ab93`，按 `F5` 试试（`CRACK` 就是要不断的试）。
6. 结果程序正确执行。试试功能，没问题。把 `48AB84` 到 `48AB8D` 的代码 `NOP` 掉，再把时间调后一年看看，依然没问题。`Cracked success?`
7. 执行菜单命令，%^&\*...，我倒，`NAG` 窗又出来了，只好继续拦截。
8. 执行万能拦窗断点，`bpx destroywindow`，`F5`，拦住。
9. `pmodule` 后，再不断的 `F12`，直到：
   ```
   0167:0048F875 648920       MOV      [FS:EAX],ESP
   0167:0048F878 803D4049490000 CMP      BYTE [00494940],00
   0167:0048F87F 7533        JNZ      0048F8B4   (NO JUMP)**在此设断 R EIP 48F8B4**
   0167:0048F881 8B0DF8394900   MOV      ECX,[004939F8]
   0167:0048F887 A1643A4900     MOV      EAX,[00493A64]
   0167:0048F88C 8B00           MOV      EAX,[EAX]
   0167:0048F88E 8B15644C4800   MOV      EDX,[00484C64]
   0167:0048F894 E8C3EDFBFF     CALL     0044E65C
   0167:0048F899 A1F8394900     MOV      EAX,[004939F8]  **在此设断 R EIP 48F8A8**
   0167:0048F89E 8B00           MOV      EAX,[EAX]
   0167:0048F8A0 8B10           MOV      EDX,[EAX]
   0167:0048F8A2 FF92D8000000   CALL     NEAR [EDX+D8]  **出现 NAG 窗**
   0167:0048F8A8 A1F8394900     MOV      EAX,[004939F8]  **TRW 将在此中断**
   0167:0048F8AD 8B00           MOV      EAX,[EAX]
   0167:0048F8AF E8A436F7FF     CALL     00402F58
   0167:0048F8B4 A1C8484900     MOV      EAX,[004948C8]
   0167:0048F8B9 8B804C010000   MOV      EAX,[EAX+014C]
   0167:0048F8BF 8945DC         MOV      [EBP-24],EAX
   0167:0048F8C2 33C0           XOR      EAX,EAX
   0167:0048F8C4 8945E0         MOV      [EBP-20],EAX
   0167:0048F8C7 8D85DCFEFFFF   LEA      EAX,[EBP+FFFFFEDC]
   0167:0048F8CD 8945E4         MOV      [EBP-1C],EAX
   0167:0048F8D0 B898FA4800     MOV      EAX,0048FA98
   0167:0048F8D5 8945E8         MOV      [EBP-18],EAX
   0167:0048F8D8 C745EC01000000 MOV      DWORD [EBP-14],01
   0167:0048F8DF 33C0           XOR      EAX,EAX
   0167:0048F8E1 8945F0         MOV      [EBP-10],EAX
   0167:0048F8E4 8D45DC         LEA      EAX,[EBP-24]
   0167:0048F8E7 50             PUSH     EAX
   0167:0048F8E8 E8C300FCFF     CALL     `SHELL32!SHBrowseForFolderA`
   0167:0048F8ED 85C0           TEST     EAX,EAX
   0167:0048F8EF 0F847F010000   JZ       NEAR 0048FA74
   0167:0048F8F5 8D95DCFEFFFF   LEA      EDX,[EBP+FFFFFEDC]
   0167:0048F8FB 52             PUSH     EDX
   0167:0048F8FC 50             PUSH     EAX
   0167:0048F8FD E8B600FCFF     CALL     `SHELL32!SHGetPathFromIDListA`
   0167:0048F902 8D45FC         LEA      EAX,[EBP-04]
   0167:0048F905 8D95DCFEFFFF   LEA      EDX,[EBP+FFFFFEDC]
   0167:0048F90B B900010000     MOV      ECX,0100
   0167:0048F910 E83B45F7FF     CALL     00403E50
   0167:0048F915 8B45FC         MOV      EAX,[EBP-04]
   0167:0048F918 E88345F7FF     CALL     00403EA0
   0167:0048F91D 85C0           TEST     EAX,EAX
   0167:0048F91F 0F8E4F010000   JNG      NEAR 0048FA74
   0167:0048F925 8B83FC030000   MOV      EAX,[EBX+03FC]
   0167:0048F92B 8BB054020000   MOV      ESI,[EAX+0254]
   0167:0048F931 8BC6           MOV      EAX,ESI
   0167:0048F931 ...            ......
   ```
10. 在 `48F87F` 处设断，强行跳过试试。成功了，没有 `NAG` 窗了。再在 `48F899` 处设一下 `R eip 48F8A8` 试试，也跳过去了，再仔细看看 `48F899` 处的代码，与 `48AB84` 处比一下，一模一样。
11. 把该处代码改后，再执行菜单看，结果---我倒，还有 `NAG`。想想第 10 步，明白了吧。哈哈，用 `UltraEdit` 查找一下 `48F899` 到 `48F8A2` 处的代码，有多少处？这就是我推荐用 `UltraEdit` 的好处，替换吧。再执行看。这次应该没问题了吧，再有问题我就$%%$%$#@....

**结论**：因为在 `48ab93` 中断时，用户屏幕没有变化，所以可以猜测程序在上一个 `CALL` 中，只是判断程序有无注册，所以可以在其上试着跳过看看，我破解的其它程序则一般是需要不断 `CALL` 进去后找到一个判断点来决定是否出现 `NAG` 窗。

**改法**：

查找 `a1f8394900 8b00 8b10 ff92d8000000`(共有 7 处)
替换为-> `9090909090 9090 9090 909090909090`

注：本人名字有朋友提醒说发不出音来，遂决定改名为 `KanKer`（砍客）。从这次的破解来看，我想我叫砍客应该是没人有意见了吧？呵呵。

=============<完>=============

Cracked by `cqzrd`   `4dd466d0e1e0cc123afbe504da2462ca`
