
### 5. 修改消息映射指针
这里就是默认的消息映射了，很显然，下面一个空位置，无法添加我们的消息响应，因此，我们必须将他搬到另外一个位置去。由于我们已经定义好了一个区段专门来存放消息响应（按 `Alt+M` 后看到 `MessageMap` 区段地址为 `00479000`），因此将 `0040A2D4` 的值改为 `00479000`，然后将上面的消息映射用二进制复制复制到 `00479000` 的位置去，然后保存文件（如果在数据段修改后保存无效，请转到代码段选定内容保存即可）。以后我们添加消息响应只需要在这后面加就好了。

### 6. 保存窗口句柄
（5）由于我们的代码要用到 `SPYXX` 获取到的窗口句柄，但是很可惜，`SPYXX` 并没有将句柄作为一个全局变量保存起来，因此我们必须自己将他保存到一个地方。下断点 `bp WindowFromPoint`,查找一个窗口后断下在如下的地方

```
004393CA  |.  50             PUSH EAX                            ; /pPoint  
004393CB  |.  FF76 20        PUSH DWORD PTR DS:[ESI+20]          ; |hWnd  
004393CE  |.  FF15 681A4000  CALL DWORD PTR DS:[<&USER32.ClientToScre>; \ClientToScreen  
004393D4  |.  FF75 10        PUSH DWORD PTR SS:[EBP+10]          ; /pt.Y  
004393D7  |.  FF75 0C        PUSH DWORD PTR SS:[EBP+C]           ; |pt.X  
004393DA  |.  FF15 641A4000  CALL DWORD PTR DS:[<&USER32.WindowFromPo>; \WindowFromPoint  
004393E0  |.  8BF8           MOV EDI,EAX  
004393E2  |.  85FF          TEST EDI,EDI  
004393E4  |.  74 7D         JE SHORT spyxx.00439463  
```

可以看到将返回的结果保存在 `edi` 这个寄存器中，我们找到 `004394D2` 修改语句为 `JMP 00478000`(00478000 为 `chCode` 的首地址),在这里编写一个代码来保存数据。

转到 `00478000`,编写以下代码：

```
00478000   893D 16804700   MOV DWORD PTR DS:[478016],EDI  
00478006   897E 64         MOV DWORD PTR DS:[ESI+64],EDI  
00478009   C746 6C 0100000>MOV DWORD PTR DS:[ESI+6C],1  
00478010   90             NOP  
00478011  - E9 C614FCFF    JMP spyxx.004394DC  
```

这里将结果保存到 `00478016` 这个位置后就调到原来的位置继续执行。

### 7. 编写消息响应
（6）下面就来编写我们的消息响应了，使用 `dd 479000`,来到全为 0 的地方，修改为以下的数据

```
00479078  00000111  
0047907C  00000000  
00479080  000038D6  
00479084  000038D6  
00479088  00000038  
0047908C  0047801F  spyxx.0047801F（可用）  
00479090  00000111  
00479094  00000000  
00479098  000038D7  
0047909C  000038D7  
004790A0  00000038  
004790A4  00478045  spyxx.00478045（变灰）  
004790A8  00000111  
004790AC  00000000  
004790B0  000038D8  
004790B4  000038D8  
004790B8  00000038  
004790BC  0047806B  spyxx.0047806B（显示）  
004790C0  00000111  
004790C4  00000000  
004790C8  000038D9  
004790CC  000038D9  
004790D0  00000038  
004790D4  0047808F  spyxx.0047808F（隐藏）  
004790D8  00000111  
004790DC  00000000  
004790E0  000038DA  
004790E4  000038DA  
004790E8  00000038  
004790EC  004780B1  spyxx.004780B1（关闭）  
```

来到 `04701F` 编写以下代码：

```
0047801F   55             PUSH EBP                    ; 可用  
00478020   8BEC           MOV EBP,ESP  
00478022   A1 16804700    MOV EAX,DWORD PTR DS:[478016]  
00478027   83F8 00        CMP EAX,0  
0047802A   74 09          JE SHORT spyxx.00478035  
0047802C   6A 01          PUSH 1  
0047802E   50             PUSH EAX  
0047802F   FF15 74136874  CALL DWORD PTR DS:[<&USER32.EnableWindow>] ; user32.EnableWindow  
00478035   C9             LEAVE  
00478036   C3             RETN  
00478037   0000           ADD BYTE PTR DS:[EAX],AL  
00478039   0000           ADD BYTE PTR DS:[EAX],AL  
0047803B   0000           ADD BYTE PTR DS:[EAX],AL  
0047803D   0000           ADD BYTE PTR DS:[EAX],AL  
0047803F   0000           ADD BYTE PTR DS:[EAX],AL  
00478041   0000           ADD BYTE PTR DS:[EAX],AL  
00478043   0000           ADD BYTE PTR DS:[EAX],AL  
00478045   55             PUSH EBP                    ; 变灰  
00478046   8BEC           MOV EBP,ESP  
00478048   A1 16804700    MOV EAX,DWORD PTR DS:[478016]  
0047804D   83F8 00        CMP EAX,0  
00478050   74 09          JE SHORT spyxx.0047805B  
00478052   6A 00          PUSH 0  
00478054   50             PUSH EAX  
00478055   FF15 74136874  CALL DWORD PTR DS:[<&USER32.EnableWindow>] ; user32.EnableWindow  
0047805B   C9             LEAVE  
0047805C   C3             RETN  
0047805D   0000           ADD BYTE PTR DS:[EAX],AL  
0047805F   0000           ADD BYTE PTR DS:[EAX],AL  
00478061   0000           ADD BYTE PTR DS:[EAX],AL  
00478063   0000           ADD BYTE PTR DS:[EAX],AL  
00478065   0000           ADD BYTE PTR DS:[EAX],AL  
00478067   0000           ADD BYTE PTR DS:[EAX],AL  
00478069   0000           ADD BYTE PTR DS:[EAX],AL  
0047806B   55             PUSH EBP                    ; 显示  
0047806C   8BEC           MOV EBP,ESP  
0047806E   A1 16804700    MOV EAX,DWORD PTR DS:[478016]  
00478073   83F8 00        CMP EAX,0  
00478076   74 09          JE SHORT spyxx.00478081  
00478078   6A 05          PUSH 5  
0047807A   50             PUSH EAX  
0047807B   FF15 78136874  CALL DWORD PTR DS:[<&USER32.ShowWindow>] ; user32.ShowWindow  
00478081   C9             LEAVE  
00478082   C3             RETN  
00478083   0000           ADD BYTE PTR DS:[EAX],AL  
00478085   0000           ADD BYTE PTR DS:[EAX],AL  
00478087   0000           ADD BYTE PTR DS:[EAX],AL  
00478089   0000           ADD BYTE PTR DS:[EAX],AL  
0047808B   0000           ADD BYTE PTR DS:[EAX],AL  
0047808D   0000           ADD BYTE PTR DS:[EAX],AL  
0047808F   55             PUSH EBP                    ; 隐藏  
00478090   8BEC           MOV EBP,ESP  
00478092   A1 16804700    MOV EAX,DWORD PTR DS:[478016]  
00478097   83F8 00        CMP EAX,0  
0047809A   74 09          JE SHORT spyxx.004780A5  
0047809C   6A 00          PUSH 0  
0047809E   50             PUSH EAX  
0047809F   FF15 78136874  CALL DWORD PTR DS:[<&USER32.ShowWindow>] ; user32.ShowWindow  
004780A5   C9             LEAVE  
004780A6   C3             RETN  
004780A7   0000           ADD BYTE PTR DS:[EAX],AL  
004780A9   0000           ADD BYTE PTR DS:[EAX],AL  
004780AB   0000           ADD BYTE PTR DS:[EAX],AL  
004780AD   0000           ADD BYTE PTR DS:[EAX],AL  
004780AF   0000           ADD BYTE PTR DS:[EAX],AL  
004780B1   55             PUSH EBP                    ; 关闭  
004780B2   8BEC           MOV EBP,ESP  
004780B4   A1 16804700    MOV EAX,DWORD PTR DS:[478016]  
004780B9   83F8 00        CMP EAX,0  
004780BC   74 0D          JE SHORT spyxx.004780CB  
004780BE   6A 00          PUSH 0  
004780C0   6A 00          PUSH 0  
004780C2   6A 10          PUSH 10  
004780C4   50             PUSH EAX  
004780C5   FF15 70136874  CALL DWORD PTR DS:[<&USER32.SendMessageA>] ; user32.SendMessageA  
004780CB   C9             LEAVE  
004780CC   C3             RETN  
```

### 8. 总结
（7）到这里，我们终于完成了全部消息响应的添加工作，保存修改后，按钮就可以正常工作了。

最后简单的总结一下，修改 `MFC` 的消息处理的关键是添加自己的消息处理，这个做起来比 `Win32` 的直接修改 `WndProc` 要复杂得多，不过可以借此了解一下 `MFC` 的内部机制，限于本人水平，里面有太多的错误的地方，还望各位大侠的批评指正。

### 附件
上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `spyxx.rar` |
| :--- | :--- |
