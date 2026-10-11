
```
  01013296    53                  PUSH EBX  
  01013297    FF95 0D0F0000       CALL DWORD PTR SS:[EBP+F0D]                 ; LoadLibrary??  
  0101329D    8985 A1050000       MOV DWORD PTR SS:[EBP+5A1],EAX             ; [ebp+5A1] = lib handle  
  .....  
  01013379   ^\E9 2FFFFFFF       JMP NOTEPAD.010132AD                        ; 跳到读取下一个函数  
  0101337E    8906              MOV DWORD PTR DS:[ESI],EAX  
  01013380    8946 0C           MOV DWORD PTR DS:[ESI+C],EAX  
  01013383    8946 10           MOV DWORD PTR DS:[ESI+10],EAX  
  01013386    83C6 14           ADD ESI,14  
  01013389    8B95 7D040000     MOV EDX,DWORD PTR SS:[EBP+47D]  
  0101338F   ^ E9 E8FEFFFF       JMP NOTEPAD.0101327C                        ; ...............................................  
```

步骤基本如下：

1. `esi` 指向源程序的 `IMAGE_IMPORT_DIRECTORY`
2. 使用 `GetModuleHandle` 或 `LoadLibrary` 获得 `DLL` 的 `handle`
3. 最后程序破坏了 `IMAGE_IMPORT_DIRECTORY` 的 `OriginFirstThunk` 等信息，应该是用来防止 `Dump` 的。不过这基本没有用处。

之后是对文件头的一些修改，因为牵扯到 `DOS` 头部，有些东西还没弄明白。

然后便是

```
0101340A    61                  POPAD  
0101340B    75 08               JNZ SHORT NOTEPAD.01013415                 ; jmp to OEP  
```

这两句恢复了寄存器数据，并且 `JMP` 到 `OEP`

至此解压完毕，从刚才的过程中我们可以获得 `IAT` 的首地址等信息来修复 `IAT`。

老规矩，放上加壳和未加壳的 `Notepad`，`UDD` 调试文件。

简单说明一下 `UDD` 的使用方法。

用 `WinHex` 等 32 位编辑器打开 `UDD` 文件，可以看到里面包含了调试文件的路径。

把它们改成你调试的 `Notepad` 的路径，然后把 `UDD` 文件考到 `OD` 的 `UDD` 目录就 `OK` 啦~~

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2009 年 08 月 31 日 23:52:52
