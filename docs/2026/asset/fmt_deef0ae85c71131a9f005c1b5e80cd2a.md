
# NetMoles Downloader Full 2.4

**标题**：NetMoles Downloader Full 2.4  
**发信人**：yimotep  
**时间**：2003/04/29 01:28am  

**详细信息**：NetMoles Downloader Full 2.4 支持`http`/`ftp`/`mms`/`mmst`/`mmsu`协议的下载软件。

- **软件性质**：共享软件
- **文件大小**：2.09M
- **最近版本**：2.4
- **操作系统**：9X/NT/
- **语言界面**：中文

## 简介

随着因特网的不断发展，网上的事物不断地丰富起来，网民对下载软件功能的要求也越来越高，NetMoles Downloader 就是为了适应这种要求而开发的。同时由于网络上的限制越来越多，NetMoles Downloader 充分利用了多代理的功能，让你轻松突破各种限制，方便下载。主要性能：捕捉在线播放的流媒体的地址（需要安装WinPcap）；支持多代理，能批量导入、导出代理，预先测试等；支持`mms`的中文文件名、有密码保护的`mms`文件的下载；将文件分成多个部分同时下载，通过多线程，断点续传等技术最大限度地提高下载速度；为了有效地管理下载的文件，可以创建不同的类别，把下载的软件分类存放；下载的任务可以根据相应的信息进行排序，查询；改进一些下载软件对同一`FTP`主机的文件下载的方法；捕获浏览器的点击、监视剪贴板、或者拖动URL到悬浮窗口；可以根据自己的时间表，计划下载。

NetMoles Downloader 是一支持`http`/`ftp`/`mms`/`mmst`/`mmsu`协议的下载软件。该软件能充分支持多代理，突破越来越多的网络限制，同时改进FlashGet和Netants等软件对同一`FTP`主机的文件下载的方法，使你更方便地下载各种软件。

目前软件的功能包括：

1. 将文件分成多个部分同时下载，通过多线程，断点续传等技术最大限度地提高下载速度。
2. 为了有效地管理下载的文件，可以创建不同的类别，把下载的软件分类存放。
3. 下载的任务可以根据相应的信息进行排序，查询。
4. 改进FlashGet和Netants等软件对同一`FTP`主机的文件下载的方法。
5. 支持`MMS`协议的中文名、有密码保护的`mms`文件等。
6. 支持多代理，以及对代理的批量导入、导出、测试等。

---

下载页面：`http://download.pchome.net/internet/download/10941.html`  
软件限制：下载20次  
作者声明：俺纯属学习，无任何目的.  
使用工具：`W32DasmV10.0`，`trw122娃娃版`，`PEiD`，`guw32`，`bw2k02`。

## 过程

由于水平有限不知算法如何，还请给位大哥指教。

先用`PEiD`看到是ASPack 2.11壳，不知用什么脱，只好听老大的用手脱，打开`bw2k02`，运行软件得到地址`005B7148`，再用`TRW122`，下`MAKEPE`，生成了新文件`NEWPE`。2M多，可是怎么也不能用，想了好长时间，用了好多方法，最后在不经意中想到了`Guw32`，感谢作者，等了若干分钟终于成功脱掉。

打开`W32DasmV10.0\参考\串式数据参考`，定位到字符串`"注册码不正确，注册失败"`。

以下是相关反汇编代码：

```
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00547458(C)
|
:005474A7 55                      push ebp
:005474A8 6842765400              push 00547642
:005474AD 64FF30                  push dword ptr fs:[eax]
:005474B0 648920                  mov dword ptr fs:[eax], esp
:005474B3 8D55F0                  lea edx, dword ptr [ebp-10]
:005474B6 8B45FC                  mov eax, dword ptr [ebp-04]
:005474B9 8B80E8020000            mov eax, dword ptr [eax+000002E8]
:005474BF E82066F0FF              call 0044DAE4
:005474C4 8B45F0                  mov eax, dword ptr [ebp-10]
:005474C7 50                      push eax
:005474C8 8D55EC                  lea edx, dword ptr [ebp-14]
:005474CB 8B45FC                  mov eax, dword ptr [ebp-04]
:005474CE 8B80E4020000            mov eax, dword ptr [eax+000002E4]
:005474D4 E80B66F0FF              call 0044DAE4
:005474D9 8B45EC                  mov eax, dword ptr [ebp-14]
:005474DC 5A                      pop edx
:005474DD E8D6950600              call 005B0AB8    <===========关键CALL ,F8进。
:005474E2 84C0                    test al, al
:005474E4 750F                    jne 005474F5      <=====jump is "Goodboy!"
\*
Possible StringData Ref from Code Obj ->"注册码不正确，注册失败"
       |
:005474E6 B858765400              mov eax, 00547658
:005474EB E86066F1FF              call 0045DB50    <======弹出错误消息
:005474F0 E92A010000              jmp 0054761F
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:005474E4(C)
|
:005474F5 33C0                    xor eax, eax
:005474F7 55                      push ebp
:005474F8 6810765400              push 00547610
:005474FD 64FF30                  push dword ptr fs:[eax]
:00547500 648920                  mov dword ptr fs:[eax], esp
:00547503 B201                    mov dl, 01
:00547505 A110EA4500              mov eax, dword ptr [0045EA10]
:0054750A E80176F1FF              call 0045EB10
:0054750F 8945F8                  mov dword ptr [ebp-08], eax
:00547512 BA02000080              mov edx, 80000002
:00547517 8B45F8                  mov eax, dword ptr [ebp-08]
:0054751A E89176F1FF              call 0045EBB0
:0054751F B101                    mov cl, 01
\*
Possible StringData Ref from Code Obj ->"\Software\NetMoles"<===如果注册码正确
         |  写入注册表这个地方
:00547521 BA78765400              mov edx, 00547678
:00547526 8B45F8                  mov eax, dword ptr [ebp-08]
:00547529 E8E676F1FF              call 0045EC14
:0054752E 8D55E8                  lea edx, dword ptr [ebp-18]
:00547531 8B45FC                  mov eax, dword ptr [ebp-04]
:00547534 8B80E4020000            mov eax, dword ptr [eax+000002E4]
:0054753A E8A565F0FF              call 0044DAE4
:0054753F 8B4DE8                  mov ecx, dword ptr [ebp-18]
\*
Possible StringData Ref from Code Obj ->"User" <======如果注册码正确注册名写入
         |\Software\NetMoles\User
:00547542 BA94765400              mov edx, 00547694
:00547547 8B45F8                  mov eax, dword ptr [ebp-08]
:0054754A E8657BF1FF              call 0045F0B4
:0054754F 8D55E4                  lea edx, dword ptr [ebp-1C]
:00547552 8B45FC                  mov eax, dword ptr [ebp-04]
:00547555 8B80E8020000            mov eax, dword ptr [eax+000002E8]
:0054755B E88465F0FF              call 0044DAE4
:00547560 8B4DE4                  mov ecx, dword ptr [ebp-1C]
\*
Possible StringData Ref from Code Obj ->"Pass"    <==如果注册码正确注册码写入
         | \Software\NetMoles\Pass
:00547563 BAA4765400              mov edx, 005476A4
:00547568 8B45F8                  mov eax, dword ptr [ebp-08]
:0054756B E8447BF1FF              call 0045F0B4
:00547570 8D55E0                  lea edx, dword ptr [ebp-20]
:00547573 8B45FC                  mov eax, dword ptr [ebp-04]
:00547576 8B80E4020000            mov eax, dword ptr [eax+000002E4]
:0054757C E86365F0FF              call 0044DAE4
:00547581 8B55E0                  mov edx, dword ptr [ebp-20]
:00547584 A18CB45B00              mov eax, dword ptr [005BB48C]
:00547589 E8DACAEBFF              call 00404068
:0054758E 8D55DC                  lea edx, dword ptr [ebp-24]
:00547591 8B45FC                  mov eax, dword ptr [ebp-04]
:00547594 8B80E8020000            mov eax, dword ptr [eax+000002E8]
:0054759A E84565F0FF              call 0044DAE4
:0054759F 8B55DC                  mov edx, dword ptr [ebp-24]
:005475A2 A198AE5B00              mov eax, dword ptr [005BAE98]
:005475A7 E8BCCAEBFF              call 00404068
:005475AC A1D4B15B00              mov eax, dword ptr [005BB1D4]
:005475B1 8B00                    mov eax, dword ptr [eax]
:005475B3 8B80B0040000            mov eax, dword ptr [eax+000004B0]
:005475B9 8B80F0010000            mov eax, dword ptr [eax+000001F0]
:005475BF BA03000000              mov edx, 00000003
:005475C4 E8EB8EF2FF              call 004704B4
:005475C9 50                      push eax
:005475CA 8B158CB45B00            mov edx, dword ptr [005BB48C]
:005475D0 8B12                    mov edx, dword ptr [edx]
:005475D2 8D45D8                  lea eax, dword ptr [ebp-28]
\*
Possible StringData Ref from Code Obj ->": 感谢您支持NetMoles Downloader"
         |
:005475D5 B9B4765400              mov ecx, 005476B4
:005475DA E81DCDEBFF              call 004042FC
:005475DF 8B55D8                  mov edx, dword ptr [ebp-28]
:005475E2 58                      pop eax
:005475E3 E8448EF2FF              call 0047042C
\* Possible
StringData Ref from Code Obj ->"注册成功, 感谢您支持NetMoles Downloader"
         |
:005475E8 B8DC765400              mov eax, 005476DC
:005475ED E85E65F1FF              call 0045DB50
:005475F2 33C0                    xor eax, eax
:005475F4 5A                      pop edx
:005475F5 59                      pop ecx
:005475F6 59                      pop ecx
:005475F7 648910                  mov dword ptr fs:[eax], edx
:005475FA 6817765400              push 00547617
```

### 关键CALL `005474DD`里的内容

```
:005B0AB8 55                      push ebp
:005B0AB9 8BEC                    mov ebp, esp
:005B0ABB 83C4F0                  add esp, FFFFFFF0
:005B0ABE 33C9                    xor ecx, ecx
:005B0AC0 894DF0                  mov dword ptr [ebp-10], ecx
:005B0AC3 8955F8                  mov dword ptr [ebp-08], edx
:005B0AC6 8945FC                  mov dword ptr [ebp-04], eax
:005B0AC9 8B45FC                  mov eax, dword ptr [ebp-04]
:005B0ACC E89339E5FF              call 00404464
:005B0AD1 8B45F8                  mov eax, dword ptr [ebp-08]
```
