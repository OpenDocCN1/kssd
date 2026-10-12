
# Dun Meter Millennium Pro 1.0

> 原文：[`www.pediy.com/kssd/pediy01/forum024.htm`](https://www.pediy.com/kssd/pediy01/forum024.htm)

解除 `Dun Meter Millennium Pro 1.0` 的延迟画面

`Dun Meter Millennium Pro 1.0` 是一个“能测试上网速度，并且用图形和文本建立下载速度表”的软件。你能自《软件》光盘 2000 年第三期中或从其作者主页 `http://dmmillennium.cjb.net/` 处得到。使用时无日期限制，但有延迟画面。

`Let's try to get rid of the NAG.`

用 `Typ_d32` 发现此软件用 `ASPACK 1.083` 压缩。首先

## 第一步：解压缩

用 `TRW2000 LOAD DmMillennium.exe`，程序自动停在入口

```
0167：004D0000    PUSHA
0167：004D0001    CALL 004DE006 <---用 F10 带过后程序主画面就显示出来。
                                                          故 F8 追入
0167:004DE559    JNZ  004DE563 <---JUMP
0167:004DE55B    MOV  EAX, 01
0167:004DE560    RET  0C
0167:004DE563    PUSH EAX
0167:004DE564    RET             <---走过这一步,就跳到
0167:004A1C18    ......          <---开始执行解压缩后的真正的程序代码
.
.
.
.
```

所以在 `0167:004DE564 RET` 这一步，可以脱壳了。`TRW2000` 共享版未提供 `PEDUMP` 与 `MAKEPE` 这两条命令，但我有 `TRW 0.75` 版，虽然很不稳定，却有所述两条命令。所以改用 `TRW 0.75`，载入后到 `0167:004DE564 RET`，下命令 `PEDUMP`（回车），然后 `G`（回车）。你会在 `DESKTOP` 或 `TRW0.75` 目录下或 `C` 盘根目录下找到 `DUMP1.EXE` 文件，这就是解压缩后的 `DmMillennium.exe` 文件。但此时执行 `DUMP1.EXE`，它会告诉你执行了非法程序。WHY？因为它的 `Entry Point` 是 `000DE564`，所以用 `PROCDUMP 1.62`，选 `PE Editor / DUMP1.EXE`，将 `Entry Point` 由 `000DE564` 改为 `000A1C18`，即真正的程序入口处，OK! 程序执行了。

## 第二步：去除延迟画面

用 `TRW2000` 再次载入 `DUMP1.EXE`

```
//****** Program Entry Point ******
:004A1C18 55                  push ebp
:004A1C19 8BEC                mov ebp, esp
:004A1C1B 83C4F4              add esp, FFFFFFF4
:004A1C1E 53                  push ebx
:004A1C1F B818174A00          mov eax, 004A1718
:004A1C24 E8AF46F6FF          call 004062D8
:004A1C29 8B1D08384A00        mov ebx, dword ptr [004A3808]
:004A1C2F 8B03                mov eax, dword ptr [ebx]
:004A1C31 E83A68FAFF          call 00448470
:004A1C36 8B03                mov eax, dword ptr [ebx]
:004A1C38 BAA01C4A00          mov edx, 004A1CA0
:004A1C3D E85264FAFF          call 00448094
:004A1C42 8B0DE4344A00        mov ecx, dword ptr [004A34E4]
:004A1C48 8B03                mov eax, dword ptr [ebx]
:004A1C4A 8B15D4F64900        mov edx, dword ptr [0049F6D4]
:004A1C50 E83368FAFF          call 00448488
:004A1C55 8B0D64354A00        mov ecx, dword ptr [004A3564]
:004A1C5B 8B03                mov eax, dword ptr [ebx]
:004A1C5D 8B1594C24900        mov edx, dword ptr [0049C294]
:004A1C63 E82068FAFF          call 00448488
:004A1C68 8B0D9C354A00        mov ecx, dword ptr [004A359C]
:004A1C6E 8B03                mov eax, dword ptr [ebx]
:004A1C70 8B1514F44900        mov edx, dword ptr [0049F414]
:004A1C76 E80D68FAFF          call 00448488
:004A1C7B A19C354A00          mov eax, dword ptr [004A359C]
:004A1C80 8B00                mov eax, dword ptr [eax]
:004A1C82 8B10                mov edx, dword ptr [eax]
:004A1C84 FF92CC000000        call dword ptr [edx+000000CC]
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
:004A1C8A 8B03                mov eax, dword ptr [ebx]
:004A1C8C E87768FAFF          call 00448508
:004A1C91 5B                  pop ebx
:004A1C92 E8051DF6FF          call 0040399C
:004A1C97 00FF                add bh, bh
:004A1C99 FFFFFF              BYTE  3 DUP(0ffh)
```

注意到：`004A1C84 FF92CC000000 call dword ptr [edx+000000CC]`，由此延迟画面出现了，强迫程序跳过去即可。故将 `FF92CC000000` 改为 `909090909090`。

OK！

孙悟空七十二变之第二变 牛魔王

`sunwk@sohu.com`

`2000.03.15`
