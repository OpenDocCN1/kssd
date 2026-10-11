
# ReGet Junior 2.0 破解手记（二）

> 原文：[`www.pediy.com/kssd/pediy04/kanxue425.htm`](https://www.pediy.com/kssd/pediy04/kanxue425.htm)

**作者**：囚童 `[FCG][BCG]`
**课题**：解除 `ReGet Junior 2.0` 显示软件名称和版本号的 `BANNER`，做内存补丁
**背景**：`PACKED WITH ASProtect v1.2`
**下载**：`http://www.cn778.net/download/c_bexpress.zip`
**工具**：
*   `TRW2000`
*   `Ultra Edit 32`
*   `R!SC's Process Patcher v1.60`

接 `ReGet Junior 2.0` 破解手记（一）

在破解手记（一）中，我们讲了解除 `ReGet Junior 2.0` 广告动画画面的两种方法。

为便于制作内存补丁，再来介绍一下第三种方法：

### 解除 `ReGet Junior 2.0` 广告动画画面方法三

启动 `TRW2000`，点 `OK`，`TRW2000` 在屏幕右下角生成一个小图标。

#### ① 运行与初步调试

运行 `regetjr.exe`，当显示软件名称和版本号的 `BANNER` 完整出现并稳定时，按下 `CTRL+N`，程序被 `TRW2000` 中断。

下指令：`BPX GETPROCADDRESS`

按 `F5`，程序运行并被 `TRW` 截获。（若截获不到，可退出重来。）

下指令：`BD *` `;清中断.`

下指令：`PMODULE`

回到 `regetjr` 域：

```text
0167：00429207　PUSH　DWORD 0046D2B8   
0167：0042920E　CALL　NEAR [004601F0]   
0167：00429214　XOR 　EDI,EDI   
0167：00429214　XOR 　EDI,EDI   
0167：00429216　MOV 　[ESI+28],EAX   
0167：00429219　CMP 　EAX,EDI   
0167：0042921B　JZ 　0042923B   
0167：0042921D　MOV 　EBX,[004600F0]   
0167：00429223　PUSH　DWORD 0046D2AC   
0167：00429228　PUSH　EAX   
0167：00429229　CALL　EBX   
0167：0042922B　PUSH　DWORD 0046D29C　〈== 光标停在这里   
0167：00429230　MOV 　[ESI+30],EAX   
0167：00429233　PUSH　DWORD [ESI+28]   
0167：00429236　CALL　EBX   
0167：00429238　MOV 　[ESI+2C],EAX   
```

下指令：`BD *` `;清中断.`

#### ② 定位广告动画代码

按 `F10`，一路来到：

```text
0167：004292DA　PUSH　EDI   
0167：004292DB　PUSH　EDI   
0167：004292DC　PUSH　BYTE +04   
0167：004292DE　PUSH　DWORD [ESI+24]   
0167：004292E1　PUSH　DWORD [ESI+20]   
0167：004292E4　PUSH　EDI   
0167：004292E5　PUSH　EDI   
0167：004292E6　PUSH　DWORD [ESI+04]   
0167：004292E9　PUSH　EDI   
0167：004292EA　PUSH　BYTE +01   
0167：004292EC　PUSH　DWORD E5　〈== 注意这个值   
0167：004292F1　CALL　EAX   
0167：004292F3　TEST　EAX,EAX   
```

下指令：`CODE ON`

记下这里的代码：`4292ED：E5`

下指令：`E CS:4292ED 0`

按 `F5`，程序运行，好，广告动画画面消失。

### 解除 `ReGet Junior 2.0` 显示软件名称和版本号的 `BANNER`

#### ③ 定位 BANNER 代码

重复上述过程 ①

显示软件名称和版本号的 `BANNER` 出现在屏幕上，并被 `TRW` 截获。（若截获不到，可退出重来。）

按 `F12`，直到显示软件名称和版本号的 `BANNER` 从屏幕上消失，我们来到：

```text
0167：0042949F　AND 　DWORD [EBP-04],BYTE +00   
0167：004294A3　MOV 　ECX,EBX   
0167：004294A5　CALL　0042D7FD　　〈== 设断   
0167：004294AA　AND 　DWORD [EDI],BYTE +00　〈== 光标停在这里   
0167：004294AD　JMP 　0042957C   
```

#### ④ 深入跟踪

下指令：`BPX 4294A5`

按 `F5` 退出。

重新运行 `regetjr.exe`，当显示软件名称和版本号的 `BANNER` 尚未出现时，按下 `CTRL+N`。

反复按 `CTRL+F5`、`CTRL+N`，直到上述代码出现在 `TRW` 代码窗口中。

此时光标停留在设断一行上。

按 `F8` 跟进，来到：

```text
0167：0042D7FD　PUSH　EBP　〈== 光标停在这里   
0167：0042D7FE　MOV 　EBP,ESP   
0167：0042D800　SUB 　ESP,EC   
0167：0042D806　PUSH　EBX   
0167：0042D807　MOV 　EBX,ECX   
0167：0042D809　PUSH　ESI   
0167：0042D80A　PUSH　EDI   
0167：0042D80B　MOV 　EAX,[EBX+04]   
0167：0042D80E　LEA 　EDI,[EBX+04]   
0167：0042D811　XOR 　ESI,ESI   
0167：0042D813　MOV 　[004772B8],EAX   
0167：0042D818　CMP 　[004658B0],ESI   
0167：0042D81E　JNZ 　0042D82B   
```

按 `F10`，一路来到：

```text
0167：0042DB2A　MOV 　EAX,[EBP-10]   
0167：0042DB2D　PUSH　DWORD 03E8   
0167：0042DB32　PUSH　EAX   
0167：0042DB33　MOV 　ECX,[EAX]   
0167：0042DB35　CALL　NEAR [ECX+64]　〈== 光标停在这里   
0167：0042DB38　MOV 　EAX,[EBP-08]   
0167：0042DB3B　CMP 　EAX,ESI   
0167：0042DB3D　JZ 　0042DB45   
```

按 `F8` 跟进，来到：

```text
0167：121A3DC0　SUB 　ESP,BYTE +1C   
0167：121A3DC3　PUSH　ESI   
0167：121A3DC4　PUSH　EDI   
0167：121A3DC5　MOV 　EDI,[ESP+2C]   
0167：121A3DC9　TEST　EDI,EDI   
0167：121A3DCB　JNL 　121A3DDA　〈== 注意这里   
0167：121A3DCD　POP 　EDI   
0167：121A3DCE　MOV 　EAX,80070057   
0167：121A3DD3　POP 　ESI   
0167：121A3DD4　ADD 　ESP,BYTE +1C   
0167：121A3DD7　RET 　08   
```

下指令：`CODE ON`

记下这里的代码：`121A3DCB：7D 0D`

下指令：`E CS:121A3DCB 90 90`

按 `F5`，程序运行，好，显示软件名称和版本号的 `BANNER` 不再在屏幕上出现。

#### ⑤ 制作内存补丁

根据上面收集到的数据，我们可以利用 `R!SC's Process Patcher v1.60` 制作一个内存补丁。

其脚本如下：

```ini
;ReGetJR 补丁程序
;撤除广告动画画面及显示软件名称和版本号的 BANNER
;loadregetjr.rpp
T=7000：               ;投放次数
F=REGETJR.EXE：        ;修补对象
O=loadregetjr.EXE：     ;输出文件
P=121A3DCB/7D,0D/90,90：;替换码
P=4292ED/E5/00：       ;替换码
$                      ;终止
```

将脚本载入 `R!SC's Process Patcher v1.60` 中，按 `SAVE AND BUILD SCRIPT` 按钮。

输出文件 `loadregetjr.EXE` 自动生成。

将生成的输出文件 `loadregetjr.EXE` 存放在与 `regetjr.exe` 同一个目录中。

运行 `loadregetjr.EXE`，程序启动较慢，但不再出现显示软件名称和版本号的 `BANNER`，广告动画画面也从窗口中消失。

解除 `ReGet Junior 2.0` 显示软件名称和版本号的 `BANNER`，消除 `ReGet Junior 2.0` 广告动画画面，补丁制作的任务到此完成。

相关文章：`ReGet Junior 2.0` 破解手记（一） `ReGet Junior 2.0` 破解手记（二） `ReGet Junior 2.0` 破解手记（三）
