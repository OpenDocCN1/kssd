
## 调用表处理

这个DLL没有跳转表的处理。其实下面的跳转表修改方法是相似。

```
代码:
---
7FF7F510     F2:66:AF            repne scas word ptr es:[edi]//寻找“目标”
7FF7F518     8B1F                mov ebx,dword ptr ds:[edi]
7FF7F51F     81E3 00000080       and ebx,80000000
7FF7F52A     81FB 00000080       cmp ebx,80000000
7FF7F530     0F85 BF000000       jnz 7FF7F5F5
7FF7F53B     8B1F                mov ebx,dword ptr ds:[edi]
7FF7F542     81E3 FFFFFF7F       and ebx,7FFFFFFF
7FF7F548     90                  nop
7FF7F549     90                  nop
7FF7F54A     90                  nop
7FF7F54B     90                  nop
7FF7F54C     90                  nop
//借这几个NOP用用，呵呵           改为：mov dword ptr ds:[edi],ebx ★
7FF7F548     891F                mov dword ptr ds:[edi],ebx
7FF7F54D     3B9D 22CD4200       cmp ebx,dword ptr ss:[ebp+42CD22]
7FF7F553     0F82 97000000       jb 7FF7F5F0
7FF7F586     3B9D 1ECD4200       cmp ebx,dword ptr ss:[ebp+42CD1E]
7FF7F58C     77 5D               ja short 7FF7F5EB
7FF7F593     833E FF             cmp dword ptr ds:[esi],-1
7FF7F59B     0F84 51030000       je 7FF7F8F2
7FF7F5A6     8B1B                mov ebx,dword ptr ds:[ebx]
7FF7F5D5     891E                mov dword ptr ds:[esi],ebx
7FF7F5DC     8937                mov dword ptr ds:[edi],esi
//写入重定位后的地址！NOP掉  ★
7FF7F5DE     90                  nop
7FF7F5DF     90                  nop
7FF7F5E0     90                  nop
7FF7F5E1     90                  nop
7FF7F5E2     90                  nop
7FF7F5E3     83C6 04             add esi,4
---
```

### 3、Magic Jump：避开IAT加密

`Ctrl+S` 在当前位置下搜索命令序列：

```
代码:
---
mov dword ptr ds:[edi],ecx
nop
nop
nop
nop
nop
add esi,4
---
```

找到在 `7FF80E83` 处。把其上的 `7FF80A43` 处修改为 `JMP`

```
代码:
---
7FF8094B     50                  push eax
7FF8094C     FFB5 44C24200       push dword ptr ss:[ebp+42C244]
7FF80952     FF95 BA6D4300       call dword ptr ss:[ebp+436DBA]
7FF809B7     3985 900E4300       cmp dword ptr ss:[ebp+430E90],eax
7FF809BD     75 4B               jnz short 7FF80A0A
7FF809EC     C685 9F774200 01    mov byte ptr ss:[ebp+42779F],1
7FF80A3C     80BD 9F774200 01    cmp byte ptr ss:[ebp+42779F],1
7FF80A43     0F85 59030000       jnz 7FF80DA2
//Magic Jump             当然改为 jmp 7FF80DA2 ★
…… …… 
7FF80D8A     61                  popad
7FF80D90     8B8D 97774200       mov ecx,dword ptr ss:[ebp+427797]
7FF80DA0     EB 0C               jmp short 7FF80DAE
7FF80DA7     8BC8                mov ecx,eax
7FF80DF7     39BD 22CD4200       cmp dword ptr ss:[ebp+42CD22],edi
7FF80DFD     76 10               jbe short 7FF80E0F
7FF80E04     89BD 22CD4200       mov dword ptr ss:[ebp+42CD22],edi
7FF80E26     39BD 1ECD4200       cmp dword ptr ss:[ebp+42CD1E],edi
7FF80E2C     73 34               jnb short 7FF80E62
7FF80E45     89BD 1ECD4200       mov dword ptr ss:[ebp+42CD1E],edi
7FF80E83     890F                mov dword ptr ds:[edi],ecx//呵呵，正确的函数写入正确的地址 :-)
7FF80E85     90                  nop
7FF80E86     90                  nop
7FF80E87     90                  nop
7FF80E88     90                  nop
7FF80E89     90                  nop
7FF80E8A     83C6 04             add esi,4
7FF80E92     83C7 04             add edi,4
7FF80EAC     E9 71F9FFFF         jmp 7FF80822//循环
---
```

### 4、OEP 啦

`Ctrl+S` 在当前位置下搜索命令序列：

```
代码:
---
popad
pop eax
cmp eax,-1
---
```

找到在 `7FF8A96C` 处。

我们现在已经完成了上面所说的修改，直接在 `7FF8A96C` 处下 内存访问 断点，`Shift+F9` 就离成功很近了。

```
代码:
---
7FF8A96C     61                  popad
7FF8A96D     58                  pop eax
7FF8A96E     83F8 FF             cmp eax,-1
7FF8A971     75 05               jnz short 7FF8A978
7FF8A973     33C0                xor eax,eax
7FF8A975     C2 0C00             retn 0C
7FF8A978     FFE0                jmp eax; EdrLib.003B11C9//飞向光明之巅！ ^O^
---
```

```
代码:
---
003B11C9     55                  push ebp//OEP ★
003B11CA     8BEC                mov ebp,esp
003B11CC     53                  push ebx
003B11CD     8B5D 08             mov ebx,dword ptr ss:[ebp+8]
003B11D0     56                  push esi
003B11D1     8B75 0C             mov esi,dword ptr ss:[ebp+C]
003B11D4     57                  push edi
003B11D5     8B7D 10             mov edi,dword ptr ss:[ebp+10]
003B11D8     85F6                test esi,esi
003B11DA     75 09               jnz short EdrLib.003B11E5
---
```

用 `LordPE` 选中 `Ollydbg` 的 `loaddll.exe` 的进程，在下面的列表里选择 `EdrLib.dll`，然后完整脱壳，得到 `dumped.dll`。

## 三、修复输入表、PE修正、完成脱壳

随便从程序找一个API调用，如：

`003B10FD     FF15 20403B00       call dword ptr ds:[3B4020]; kernel32.GetVersion`

避开重定位后是 `call dword ptr ds:[404020]`，你可以改为 `[3B4020]` 就能看到函数调用了。

在转存中跟随 `3B4020`，上下看到许多函数地址，很明显的可以找到IAT开始和结束的地址：

开始地址＝`003B4000`

结束地址＝`003B40C9`

运行 `ImportREC`，注意：去掉“使用来自磁盘的PE部首”的选项！

选中 `Ollydbg` 的 `loaddll.exe` 的进程，然后点“选取DLL”，选择 `EdrLib.dll`，填入 `RVA＝00004000`、`大小＝000000C8` ，点“Get Import”，得到输入表。改 `OEP=000011C9`，`FixDump`！

用 `LordPE` 修正 `dumped\_.dll` 的重定位表 `RVA=00006000`、`大小＝000003B0`，保存之。

因为在 `7FF80FC4` 处跳过了重定位，所以不需要修正基址。OK，脱壳完成啦。

可以用 `LordPE` 删除最后一个区段，去掉“脱壳修复”和“清除重定位表”的选项简单优化一下脱壳后的DLL。

```
代码:
---
          ,     _/ 
         /| _.-~/            \_     ,        青春都一晌
        ( /~   /              \~-._ |\
        `\\  _/                \   ~\&
    _-~~~-.)  __/;;,.          \_  //'
   /'_,\   --~   \ ~~~-  ,;;\___(  (.-~~~-.        换了破解轻狂
  `~ _( ,_..--\ (     ,;'' /    ~--   /._`\ 
   /~~//'   /' `~\        
   "  `~"  "      `"      /~'`\    `\\~~\   
                          "     "   "~'  ""
---
```

Cracked By 巢水工作坊――fly [OCN][FCG][NUKE][DCM]

2004-06-06  19:00
