
# MicroAngelo 98 4.77

> 原文：[`www.pediy.com/kssd/pediy02/forum145.htm`](https://www.pediy.com/kssd/pediy02/forum145.htm)

桌面工具 | ICON 工具

http://www.newhua.com.cn/Microangelo98.htm

软件名称：MicroAngelo 98

整理日期：2000.4.10

最新版本：4.77

文件大小：1361KB

软件授权：共享软件

使用平台：Win95/98/NT

发布公司：http://www.impactsoft.com/index.htm

软件简介：

MicroAngelo 是一个可编辑和管理光标的工具，可让您轻易地浏览、管理和编辑 `ICO`、`ICL`、`EXE`、`DLL` 文件中的图示文件，支持单色，16 或者 256 颜色及 8x8 到 64x64 的图素尺寸。还能调整设定你的桌面，如布景主题、移除桌面图示捷径的箭头等。说明文章

本地下载：

http://www.newhua.com/down/mike98.exe

http://www.win2000.com.cn/down/mike98.exe

国外下载：

http://www.impactsoft.com/muangelo/download/mike98.exe

## 破解过程

使用 `BPX GetLocalTime`，按两次 `F12`，然后按 `F10`，你将看到所有的时间检查。它使用诸如 `"State"`、`"LastRun"`、`"CheckNum"` 等注册表键。删除这些键并重新安装。

反汇编 `MUAPP.DLL`，读取其导出函数。函数名称不言自明。

修补其 `EXE` 文件是个好主意。

这个软件（包）属于比较简单的那一类。先用 `RegMon`、`FileMon` 找一下，看见几个可疑的注册表键：`Start`、`State`、`CheckNum` 等。以 `Animator` 为例，用 `bpx RegQueryValueExA do "d *(esp+8)"` 设断点，看见它读 `Start` 键的时候就改按 `F12` 或 `F10`，会看见它检查注册表中的这几个键。可见，只要 `DS:[0042BD60]` 为非零值，它就不会检查时间了。于是用 `BPM 0042BD60 W` 设个断点，发现它把 `LoadLibraryA("MUCOM.DLL",...)` 的返回值放在这个变量中，这说明只要存在一个有效的 `mucom.dll` 就是注册版，因为注册版不用检查日期，且不会在窗口的标题栏中显示“Evaluation day 1 of 30”之类的鬼话，而且它还会调用这个 dll 中的 `GetCopyRight( )` 显示注册用户的信息。由于无法构造一个有效的 `mucom.dll`，故只需要把下面的第一个 `JNZ` 改为 `JMP` 即可。注意：不要朝 `[0042BD60]` 中写入非零值，因为它是 DLL 的句柄，弄不好会出非法操作。其它的几个 `EXE` 与此类似。

```
015F:0040C3ED  MOV       EAX,[0042BD60]
015F:0040C3F2  TEST      EAX,EAX
015F:0040C3F4  JNZ       0040C40C
015F:0040C3F6  CALL      004184C0    //这里头检查时间。
015F:0040C3FB  TEST      EAX,EAX
015F:0040C3FD  JNZ       0040C40C
015F:0040C3FF  POP       EDI
015F:0040C400  POP       ESI
015F:0040C401  POP       EBP
015F:0040C402  POP       EBX
015F:0040C403  ADD       ESP,000001E4
015F:0040C409  RET       0008
015F:0040C40C  MOV       EAX,[0042BE8C]
```

至于 `muapp.dll` 的 `CRC` 校验，用 `bpx CreateFileA` 容易搞定。只要是 `CRC` 校验，一定要使用文件读写或内存映射文件的 API，用这些 API 作断点即可。共两处 `CRC` 校验，第一处是检查自己，第二处是检查 `EXE`。似乎在 `DllMain( )` 中干的，因为说无法启动该 DLL。

```
015F:6400135A  89442414            MOV       [ESP+14],EAX
015F:6400135E  85DB                TEST      EBX,EBX
015F:64001360  7FCB                JG        6400132D
015F:64001362  56                  PUSH      ESI
015F:64001363  FF157C700064        CALL      [KERNEL32!CloseHandle]
015F:64001369  A130900064          MOV       EAX,[64009030]
015F:6400136E  8B542414           MOV       EDX,[ESP+14]
015F:64001372  3BD0                CMP       EDX,EAX      //第一处
015F:64001374  0F85BDFCFFFF        JNZ       64001037

.........................

015F:6400156B  85C0                TEST      EAX,EAX
015F:6400156D  7507                JNZ       64001576
015F:6400156F  B803000000          MOV       EAX,00000003
015F:64001574  EB19                JMP       6400158F
015F:64001576  8D542430            LEA       EDX,[ESP+30]
015F:6400157A  681C910064          PUSH      6400911C
015F:6400157F  52                  PUSH      EDX
015F:64001580  FFD6                CALL      ESI
015F:64001582  85C0                TEST      EAX,EAX
015F:64001584  0F85ADFAFFFF        JNZ       64001037
015F:6400158A  B801000000          MOV       EAX,00000001
015F:6400158F  8B4C2414            MOV       ECX,[ESP+14]
015F:64001593  8B148534900064      MOV       EDX,[EAX*4+64009034]
015F:6400159A  3BCA                CMP       ECX,EDX      //第二处
015F:6400159C  0F8595FAFFFF        JNZ       64001037
015F:640015A2  5F                  POP       EDI
015F:640015A3  5E                  POP       ESI
015F:640015A4  5D                  POP       EBP
015F:640015A5  B801000000          MOV       EAX,00000001  //good guy
015F:640015AA  5B                  POP       EBX
015F:640015AB  81C4B8020000        ADD       ESP,000002B8
015F:640015B1  C20C00              RET       000C
```

Comparing files `muapp.dll`

```
000011B3: 74 EB
0000136F: 54 D0
00001370: 24 90
00001371: 14 90
00001C48: 75 EB
000024DE: 75 90
000024DF: 0F 90
00002835: 85 B8
00002836: C0 01
00002837: 74 00
00002838: 05 00
00002839: 33 00
0000283A: C0 90
0008893E: 28 00
000889CC: 25 00   4dd466d0e1e0cc123afbe504da2462ca
```
