
#### 1. 00FF17ED

```
00FF17ED   8B0D 8464FF00    MOV ECX,DWORD PTR DS:[FF6484]

00FF17F3   8B09             MOV ECX,DWORD PTR DS:[ECX]

00FF17F5   3BC8             CMP ECX,EAX

00FF17F7   75 09            JNZ SHORT 00FF1802

00FF17F9   8B0495 D863FF00  MOV EAX,DWORD PTR DS:[EDX*4+FF63D8]

00FF1800   EB 07            JMP SHORT 00FF1809

00FF1802   52               PUSH EDX

00FF1803   50               PUSH EAX

00FF1804   E8 B739FFFF      CALL 00FE51C0     ; JMP to kernel32.GetProcAddress

00FF1809   5D               POP EBP

00FF180A   C2 0800          RETN 8
```

正如我们看到的那样: `GetProcAddress` <-- Good `Imprec` !

-----------------------------------------------------------------------------------

#### 2. 00FF1CA4

```
00FF1CA4   55               PUSH EBP

00FF1CA5   8BEC             MOV EBP,ESP

00FF1CA7   8B45 08          MOV EAX,DWORD PTR SS:[EBP+8]

00FF1CAA   85C0             TEST EAX,EAX

00FF1CAC   75 13            JNZ SHORT 00FF1CC1

00FF1CAE   813D A47AFF00 00>CMP DWORD PTR DS:[FF7AA4],400000   ; ASCII "MZP"

00FF1CB8   75 07            JNZ SHORT 00FF1CC1

00FF1CBA   A1 A47AFF00      MOV EAX,DWORD PTR DS:[FF7AA4]

00FF1CBF   EB 06            JMP SHORT 00FF1CC7

00FF1CC1   50               PUSH EAX

00FF1CC2   E8 F134FFFF      CALL 00FE51B8     ; JMP to kernel32.GetModuleHandleA

00FF1CC7   5D               POP EBP

00FF1CC8   C2 0400          RETN 4

00FF1CCB   90               NOP

00FF1CCC   FF35 147EFF00    PUSH DWORD PTR DS:[FF7E14]

00FF1CD2   58               POP EAX

00FF1CD3   C3               RETN
```

如上: `GetModuleHandleA` <-- Good `Imprec` !

-----------------------------------------------------------------------------------

#### 3. 00FF1D18

```
00FF1D18   6A 00            PUSH 0

00FF1D1A   E8 9934FFFF      CALL 00FE51B8     ; JMP to kernel32.GetModuleHandleA

00FF1D1F   FF35 147EFF00    PUSH DWORD PTR DS:[FF7E14]

00FF1D25   58               POP EAX

00FF1D26   8B05 247EFF00    MOV EAX,DWORD PTR DS:[FF7E24]

00FF1D2C   C3               RETN
```

仔细看看，你会发现这个可不是 `GetModuleHandleA`

为什么呢? 因为在 `API` `GetModuleHandleA` 的调用之后程序利用这段代码来获取某些信息存入 `EAX` 作为调用的返回值 ..

那么这个是是什么函数呢? 执行完这段代码之后你会发现 `EAX` 中是程序路径的 `ascii` 字符串。 所以这个是: `GetCommandLineA` <-- Good `Imprec` !

-----------------------------------------------------------------------------------

#### 4. 00FF1D08

```
00FF1D08   55               PUSH EBP

00FF1D09   8BEC             MOV EBP,ESP

00FF1D0B   8B05 247EFF00    MOV EAX,DWORD PTR DS:[FF7E24]

00FF1D11   8B45 08          MOV EAX,DWORD PTR SS:[EBP+8]

00FF1D14   5D               POP EBP

00FF1D15   C2 0400          RETN 4
```

仔细瞧瞧 .. 这个函数在开始的时候 `push` `EBP` 然后在结束的时候 `Pop` `EBP`, 所以这个函数什么都没执行，应该是 `LockResource` 或者 `FreeResource`

我选择的是: `FreeResource` <-- Good `Imprec` !

-----------------------------------------------------------------------------------

#### 5. 00FF1D00

```
00FF1D00   A1 207EFF00      MOV EAX,DWORD PTR DS:[FF7E20]

00FF1D05   C3               RETN
```

这个真有趣 .. 执行这段代码你会看到在 `EAX` 中有一个值， 这个值是进程的 `PID`!

打开 `ImpRec` 在进程列表中你会看到同样的 `PID` 值(`PID` - `Process` `ID`)

所以这个是 : `GetCurrentProcessId` <-- Good `Imprec` !

-----------------------------------------------------------------------------------

#### 6. 00FF1CF8

```
00FF1CF8   A1 187EFF00      MOV EAX,DWORD PTR DS:[FF7E18]

00FF1CFD   C3               RETN
```

在：`00FF1CF8` 和 `00FF1D00` 这两个邻近的地址中我只能在 `00FF1D00` 这个地址设置断点，发现是: `GetCurrentProcessId`

我只能得出一个结论 .. 这个 `api` 一定是: `GetCurrentProcess`

新版本的 `imprec` 也把这个地址修复为 `GetCurrentProcessId`...

在新版本的 `Imprec` 的 `News.txt` 中说明了:

"出色的 `disassembler`/`hew-viewer` 功能有助于你查看重定向代码. 你不再需要 `ASProtect` 修复插件，因为 `ASProtect` 修复插件并没有更新，只是做为一个例子放在那儿"

问题不是出在 `ImpRec` 而是那个插件，我说这些只是为了你能有一些了解。

.. `GetCurrentProcessId` 也工作的很好...

-----------------------------------------------------------------------------------

#### 7. 00FF1D30

```
00FF1D30   55               PUSH EBP

00FF1D31   8BEC             MOV EBP,ESP

00FF1D33   8B05 247EFF00    MOV EAX,DWORD PTR DS:[FF7E24]

00FF1D39   5D               POP EBP

00FF1D3A   C2 0400          RETN 4
```

如前所叙，我选的是   : `FreeResource`    <-- Good `Imprec` !

好了，`IAT` 修复完了 ..

=============================================

### 如何修复 OEP 并补上 Stolen bytes

=============================================

用 `Pe-Editor` of `ProcDump`   `Lord-PE` 把 `OEP` 修改为:`564BDC`

然后打开 `Hex-Edit`  `Hview` 把 `OEP` 处的 `"00"` 修改为

```
Push EBP

Mov EBP,ESP

Add ESP,-010

Mov eax, 05644CC
```

到这里 `aspr` 的脱壳就完成了...

### 致谢 :

=============

to all the ppl that have helped me lern and beeing there for me ..

`Mackt` , `^Daemon^` , `VAG` , `EvilWT` , `dynm8` , `urad0x` , `parabytes` , `Jb__` , `NeOXQuiCK` , `SexyGeek` ,

`ThE-SAiNt`

`SAC` , `NchantA` , `Eternal` `Bliss` , `evaluator` , `Bengaly` , `Termin-X` , `jond` .

all the ppl at `http://woodmann.cjb.net` -   `RCE` `Messageboard's`

and to all the ppl the i forgoted :D

`4dd466d0e1e0cc123afbe504da2462ca`
