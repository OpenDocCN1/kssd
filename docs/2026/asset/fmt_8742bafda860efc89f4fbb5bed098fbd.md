
......略一段

从`00406D82` 到 `00407004` 通过`FindWindowExA`函数寻找的窗口句柄，关闭窗口名为下列的应用程序`QQKav`.`防火墙`.`网镖`.`杀毒`.`病毒`.`木马`.`恶意`.`QQAV`.`噬菌体`.这些保护程序.

```
0040711F   .  A1 F4A04000   MOV EAX,DWORD PTR DS:[40A0F4]
00407124   .  8B00          MOV EAX,DWORD PTR DS:[EAX]
00407126   .  E8 C5DBFFFF   CALL SVOHOST.00404CF0
0040712B   .  69C0 E8030000 IMUL EAX,EAX,3E8
00407131   .  50            PUSH EAX                          ; /Timeout
00407132   .  E8 29D3FFFF   CALL <JMP.&kernel32.Sleep>         ; \Sleep
00407137   .  B8 68714000   MOV EAX,SVOHOST.00407168      ;  qq.exe
0040713C   .  E8 53D9FFFF   CALL SVOHOST.00404A94
```

我们可怜的QQ也在其中，在设置的时间内会关闭QQ.还好，我用的是QQ他妈TM.

```
00407291  |.  68 287A4000   PUSH SVOHOST.00407A28
//生成一个noruns.reg文件，看样子要写注册表了
00407296  |.  8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]
00407299  |.  BA 03000000   MOV EDX,3
0040729E  |.  E8 3DCAFFFF   CALL SVOHOST.00403CE0
004072A3  |.  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]
004072A6  |.  E8 6DCBFFFF   CALL SVOHOST.00403E18
004072AB  |.  50            PUSH EAX                                 ; |Parameters
004072AC  |.  68 D87A4000   PUSH SVOHOST.00407AD8                    ; |regedit.exeopen
004072B1  |.  68 E47A4000   PUSH SVOHOST.00407AE4         ; |open
004072B6  |.  6A 00         PUSH 0                           ; |hWnd = NULL
004072B8  |.  E8 C3D3FFFF   CALL SVOHOST.00404680      ; \ShellExecuteA
004072BD  |.  6A 00         PUSH 0                        ; /IsShown = 0
004072BF  |.  6A 00         PUSH 0                       ; |DefDir = NULL
004072C1  |.  68 EC7A4000   PUSH SVOHOST.00407AEC       ; |stop srservice
004072C6  |.  68 FC7A4000   PUSH SVOHOST.00407AFC                    ; |net.execonfig srservice start= disabled
004072CB  |.  68 E47A4000   PUSH SVOHOST.00407AE4            ; |open
004072D0  |.  6A 00         PUSH 0                             ; |hWnd = NULL
004072D2  |.  E8 A9D3FFFF   CALL SVOHOST.00404680        ; \ShellExecuteA
004072D7  |.  6A 00         PUSH 0                            ; /IsShown = 0
004072D9  |.  6A 00         PUSH 0                            ; |DefDir = NULL
004072DB  |.  68 047B4000   PUSH SVOHOST.00407B04
; |config srservice start= disabled
004072E0  |.  68 287B4000   PUSH SVOHOST.00407B28            ; |sc.exe
004072E5  |.  68 E47A4000   PUSH SVOHOST.00407AE4             ; |open
004072EA  |.  6A 00         PUSH 0                               ; |hWnd = NULL
004072EC  |.  E8 8FD3FFFF   CALL SVOHOST.00404680                    ; \ShellExecuteA
```

......略一段

```
00407689  |.  68 9C7E4000   PUSH SVOHOST.00407E9C          ; /Title = "瑞星提示"
0040768E  |.  68 A87E4000   PUSH SVOHOST.00407EA8          ; |Class = "#32770"
00407693  |.  E8 10CEFFFF   CALL <JMP.&user32.FindWindowA>           ; \FindWindowA
00407698  |.  68 B07E4000   PUSH SVOHOST.00407EB0           ; /Title = "是(&Y)"
0040769D  |.  68 B87E4000   PUSH SVOHOST.00407EB8     ; |Class = "Button"
004076A2  |.  6A 00         PUSH 0                                   ; |hAfterWnd = NULL
004076A4  |.  50            PUSH EAX                                 ; |hParent
004076A5  |.  E8 06CEFFFF   CALL <JMP.&user32.FindWindowExA>         ; \FindWindowExA
```

......略一段

从`00407291` 到`00407893` 这样长的一断代码，都是杀毒，防火墙，做斗争，我就不清楚这些系统命令`Sc.EXE` `Net.exe` `net1.exe`也加入到这个木马封杀的行列.

```
004078FB  |.  6A 01         PUSH 1
004078FD  |.  B9 9C824000   MOV ECX,SVOHOST.0040829C                        ;  ASCII "SoundMam"
00407902  |.  BA A8824000   MOV EDX,SVOHOST.004082A8   //添加启动项
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
"SoundMam" = "%system%\SVOHOST.exe"
00407907  |.  B8 02000080   MOV EAX,80000002
0040790C  |.  E8 0BD8FFFF   CALL SVOHOST.0040511C
00407911  |.  68 D8824000   PUSH SVOHOST.004082D8
00407916  |.  6A 01         PUSH 1
00407918  |.  B9 DC824000   MOV ECX,SVOHOST.004082DC                        ;  ASCII "CheckedValue"
0040791D  |.  BA EC824000   MOV EDX,SVOHOST.004082EC                        ;  ASCII "software\microsoft\windows\currentversion\explorer\advanced\folder\hidden\showall"
//隐藏的系统文件
00407922  |.  B8 02000080   MOV EAX,80000002
00407927  |.  E8 F0D7FFFF   CALL SVOHOST.0040511C
0040792C  |.  8D45 E0       LEA EAX,DWORD PTR SS:[EBP-20]
0040792F  |.  E8 64D7FFFF   CALL SVOHOST.00405098
00407934  |.  8D45 E0       LEA EAX,DWORD PTR SS:[EBP-20]
```

木马的主程序的大体功能和运行机制就分析到这里，还有一个插入到IE进程的`winscok.dll`文件，我这里就跳过了。无非就是刷流量。

```
<HTML><HEAD><TITLE>.</TITLE>
<meta http-equiv="refresh" content="1; url=6b3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4S2^5P5s2S2Q4x3X3g2F1k6i4b7`."> ‘地址用xxx代替了
</HEAD><BODY>
<script src='ed6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4x3@1y4#2)9J5k6h3y4F1P5Y4A6Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6r3q4@1i4K6u0W2M7r3S2H3i4K6y4r3K9h3c8Q4x3@1b7J5x3U0x3$3z5e0N6Q4x3U0k6%4k6h3u0Q4y4h3k6A6k6q4)9K6c8o6t1J5x3K6j5&6y4#2)9J5y4H3`.`. language='JavaScript' charset='gb2312'></script>   ’站长站的流量统计
</BODY></HTML>
```

不过这马还有盗QQ，哎。什么世道啊。

分析过程就不做总结了。想必你一路看完下来也理解了。我们直接给出手工解决的办法，多动手，多学习。让你遇到这个木马不会那么头疼。毕竟做了系统，不小心又是会感染的。

### 一步：进入安全模式
我们先来显示出被隐藏的系统文件的regedit

`HKEY_LOCAL_MACHINE\Software\Microsoft\windows\CurrentVersion\explorer\Advanced\Folder\Hidden\SHOWALL`

将`CheckedValue`键值修改为`1`

![](http://bbs.pediy.com/upload/2006/4/image/image009.png)

这里要注意，病毒会把本来有效的`DWORD`值`CheckedValue`删除掉，新建了一个无效的字符串值`CheckedValue`,类型为`REG_SZ`，并且把键值改为`0`！我们将这个改为`1`是毫无作用的。没有看清楚`CheckedValue`后面的类型，正确的是`RED	DWORD`而不是`REG_SZ`。

### 二步：然后在来解决注册启动项。

`HKEY_LOCAL_MACHINE>SOFTWARE>Microsoft>Windows>CurrentVersion>Run`

`SVOHOST.exe`

### 三步：Ctrl + Alt + Del 任务管理器，在进程中查找 `SVOHOST`（不是`SVCHOST`，相差一个字母），有的话就将它结束掉（并不是所有的系统都显示有这个进程，没有的就略过此步）。

### 四步：删除系统目录的二个文件`SVOHOST.exe`和`winscok.dll`文件。我们可以在命令执行

```
DEL C:\windows\system32\SVOHOST.exe /S
DEL C:\windows\system32\winscok.dll /S
```

如果查找不到文件，记得把他的系统属性去掉。

### 五步：解决其他盘的遗留问题，我这里已经D盘为例，其他分区同样超做。CMD执行命令

```
attrib -h -r -s d:\sxs.exe
del d:\sxs.exe
attrib -h -s -r d:\autorun.inf
del d:\autorun.inf
```

解决完小马，我们来追查源头！如果访问其他分区目录，木马就会感染你的系统和其他分区，不是通过局域网，这种情况的传播途径就只有USB，不知道大家注意没有，使用U盘或者MP3，是直接运行.那！我们的小马就开始他的传播了.....

MM：怪不的我的MP3会提示有毒……….

                                                              （完）

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [别让mp3黑走你的qq.rar](javascript:void(0);)

  （154.55kb，35次下载）
