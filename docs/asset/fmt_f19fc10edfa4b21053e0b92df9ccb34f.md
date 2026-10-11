
```
  00343A7C    66:9C               pushfw  

  00343A7E    E8 04000000         call 00343A87  

  00343A83    0010               add byte ptr ds:[eax], dl  

  00343A85    40                 inc eax  

  00343A86    0083 C404EB04      add byte ptr ds:[ebx+4EB04C4], al  

  00343A8C    31C8               xor eax, ecx  

  00343A8E    3F                 aas  

  00343A8F    00EB               add bl, ch  

  00343A91    04 F0              add al, 0F0  

  00343A93    0F4000             cmovo eax, dword ptr ds:[eax]  

  00343A96    66:9D              popfw  

  00343A98    70 06              jo short 00343AA0  

  00343A9A    71 04              jno short 00343AA0  

  00343A9C    A8 C8              test al, 0C8  

  00343A9E    3F                 aas  

  00343A9F    0072 07            add byte ptr ds:[edx+7], dh  

  00343AA2    73 05              jnb short 00343AA9  

  00343AA4    880F               mov byte ptr ds:[edi], cl  

  00343AA6    E0 3E              loopdne short 00343AE6  

  00343AA8    00EB               add bl, ch  

  00343AAA    AD                 lods dword ptr ds:[esi]  

  00343AAB    74 08              je short 00343AB5  

  00343AAD    75 06              jnz short 00343AB5  

  00343AAF    9A 00E03D00 E876   call far 76E8:003DE000                ; Far call  

  00343AB6    07                 pop es                                ; Modification of segment register  

  00343AB7    77 05              ja short 00343ABE  

  00343AB9    E7 BB              out 0BB, eax                         ; I/O command  

  00343ABB    EC                 in al, dx                            ; I/O command  

  00343ABC    3E:0078 07         add byte ptr ds:[eax+7], bh  

  00343AC0    79 05              jns short 00343AC7  

  00343AC2    EA 000F41FF 669C   jmp far 9C66:FF410F00                 ; Far jump  

  00343AC9    EB 06              jmp short 00343AD1  

  00343ACB    E8 01AADB3D        call 3E0FE4D1  

  00343AD0    006A 02            add byte ptr ds:[edx+2], ch  

  00343AD3    EB 06              jmp short 00343ADB  

  00343AD5    E8 A0994700        call 007BD47A  

  00343ADA    9A 7311EB05 E80A   call far 0AE8:05EB1173               ; Far call  

  00343AE1    0029               add byte ptr ds:[ecx], ch  

  00343AE3    E9 E80C0000        jmp 003447D0  

  00343AE8    008B FF0F41FE      add byte ptr ds:[ebx+FE410FFF], cl  

  00343AEE   ^ 73 F4            jnb short 00343AE4  

  00343AF0    9A 20143100 83C4   call far C483:00311420               ; Far call  

  00343AF7    04 EB              add al, 0EB  

  00343AF9    04 D0              add al, 0D0  

  00343AFB    39C7               cmp edi, eax  

  00343AFD    00FF               add bh, bh  

  00343AFF    0C 24              or al, 24  

  00343B01    71 04              jno short 00343B07  

  00343B03    BA 88260079        mov edx, 79002688  

  00343B08    D27A 01            sar byte ptr ds:[edx+1], cl  

  00343B0B    E7 83              out 83, eax                           ; I/O command  

  00343B0D    C40466             les eax, fword ptr ds:[esi]           ; Modification of segment register  

  00343B10    9D                 popfd  

  00343B11    EB 05              jmp short 00343B18  

  00343B13    B8 EF0F410F        mov eax, 0F410FEF  

  00343B18    8B46 04            mov eax, dword ptr ds:[esi+4]   //在这里下 F2 断点  

  00343B1B    8903               mov dword ptr ds:[ebx], eax  

  00343B1D    61                 popad  

  00343B1E    58                 pop eax  

  00343B1F    8B00               mov eax, dword ptr ds:[eax]  

  00343B21    FFE0               jmp eax  

  00343B23    8B46 04            mov eax, dword ptr ds:[esi+4]   //在这里下 F2 断点  

  00343B26    8903               mov dword ptr ds:[ebx], eax  

  00343B28    61                 popad  

  00343B29    58                 pop eax  

  00343B2A    83C4 04            add esp, 4  

  00343B2D    8B00               mov eax, dword ptr ds:[eax]  

  00343B2F    FFE0               jmp eax  
```

经跟踪分析，这是查表找到对应的 API，然后由 `jmp eax` 跑到 API 执行。

当我们在两句 `mov eax, dword ptr ds:[esi+4]` 都下好 `F2` 断点后，`F9` 运行，会发现这次我们中断在了：

```
  00343B23    8B46 04            mov eax, dword ptr ds:[esi+4]
```

信息窗口内容是：

```
ds:[00343E1E]=0067535C (MTVAlbum.0067535C)
eax=00406E96 (MTVAlbum.00406E96)
```

在数据窗口跟踪 `00343E1E` 得到如下一个很有规律的对应表：

```
  00343E1A  96 6E 40 80  //这个就是在上面让大家记住的地址$$$$$$
  00343E1E  5C 53 67 00  //而这里是什么，下面就看到了
  00343E22  82 71 40 80  
  00343E26  64 53 67 00  
  00343E2A  7A 71 40 80  
  00343E2E  68 53 67 00  
  00343E32  72 71 40 80  
```

现在上下找找这张表的开始和结束的地址： `//先记下，一会写修复代码时会用到`

```
开始地址: 343C2A  
结束地址: 344EF2  
```

当 `F7` 来到 `00343B2D` 时：

```
  00343B26    8903               mov dword ptr ds:[ebx], eax  
  00343B28    61                 popad  
  00343B29    58                 pop eax  
  00343B2A    83C4 04            add esp, 4  
  00343B2D    8B00               mov eax, dword ptr ds:[eax]  
  00343B2D    8B00               mov eax, dword ptr ds:[eax]         ; KERNEL32.GetModuleHandleA  
```

信息窗口内容：

```
ds:[0067535C]=77E63DFC (KERNEL32.GetModuleHandleA)  
eax=0067535C (MTVAlbum.0067535C)  
```

在数据窗口跟踪 `0067535C` 得到一个很完整的 IAT 表。如下：

```
  0067535C  kernel32.dll  013F  GetModuleHandleA  
  00675360  00000000  
  00675364  advapi32.dll  01AF  RegQueryValueExA  
  00675368  advapi32.dll  01A5  RegOpenKeyExA  
  0067536C  advapi32.dll  018C  RegCloseKey  
  00675374  00000000  
  00675374  kernel32.dll  0335  lstrcpy  
  00675378  kernel32.dll  0332  lstrcmpi  
  0067537C  kernel32.dll  032F  lstrcmp  
```

现在上下找找 IAT 开始地址和大小： `//一会修复时用 ImportREC 取 IAT 时会用到`

```
开始地址: 343C2A  
结束地址: 344EF2  
```

当执行到：

```
  00343B2F    FFE0               jmp eax    // 跳到函数去了。  
```

下面我们来分析下这个对照表的关系：

```
  00406E90    90                 nop  
  00406E91    E8 A4CBF3FF        call 00343A3A //查表找到下面的 00406E96 地址对应的 API  
  00406E96    8BC0               mov eax, eax  //根据这里的地址来查，即 00406E96  
  00406E98    90                 nop  
```

来看看之前找到的那张表：

```
  00343E1A  96 6E 40 80  //这里是 00406E96  
  00343E1E  5C 53 67 00  //而这里就是 00406E96 对应的 API 的存放地址  
  00343E22  82 71 40 80   
  00343E26  64 53 67 00  
  00343E2A  7A 71 40 80   
  00343E2E  68 53 67 00  
  00343E32  72 71 40 80   
```

同理 `00343E22` 放的地址是 `00407182`，他所对应的 API 就放在 `00675364`。

这下子清晰了吧，这个表就是 `call 00343A3A` 根据地址来得到对应的函数的对照表。

好了，下面让我们来修复吧：

还记得上次在 OEP 下过硬件执行断点吧，这下就用上了。

`Ctrl+F2` 重新载入程序，`F9` 运行，会中断在 OEP 处。然后找个空白段写入以下代码：

这程序代码段尾部有足够的地方，就不去别处找了：

```
  00696EFF    90                 nop             //首先在这新建 EIP  
  00696F00    60                 pushad  
  00696F01    9C                 pushfd  
  00696F02    BB 2A3C3400        mov ebx, 343C2A  //对照表开始位置  
  00696F07    8B03               mov eax, dword ptr ds:[ebx]  
  00696F09    2D 00000080        sub eax, 80000000  
  00696F0E    66:C740 FA FF25    mov word ptr ds:[eax-6], 25FF  
  00696F14    8B4B 04            mov ecx, dword ptr ds:[ebx+4]  
  00696F17    8948 FC            mov dword ptr ds:[eax-4], ecx  
  00696F1A    83C3 08            add ebx, 8  
  00696F1D    81FB F24E3400      cmp ebx, 344EF2   //对照表结束位置  
  00696F23    7D 02              jge 00696F27  
  00696F25   ^ EB E0            jmp 00696F07  
  00696F27    9D                 popfd  
  00696F28    61                 popad  
  00696F29   ^ E9 9AD1FAFF       jmp 006440C8  //跳到 OEP  
  00696F2E    90                 nop  
```

二进制代码是：

```
90 60 9C BB 2A 3C 34 00 8B 03 2D 00 00 00 80 66 C7 40 FA FF 25 8B 4B 04 89 48 FC 83 C3 08 81 FB F2 4E 34 00 7D 02 EB E0 9D 61 E9 9A D1 FA FF 90
```

`F9` 运行后会再次中断在 OEP。

打开 `LordPE`，先执行一下`"correct ImageSize"`，纠正一下文件的大小，再来 `DUMP` 程序。

然后打开 `ImportREC` 选择程序 `OPE:002440C8`，然后自动查找 IAT，获取输入表。

这会得到一个完好的表了。

修复之后运行正常。

---

## 经验总结

以前脱这种壳只会补区段，在学习了 `xiaoboy` 兄弟的“`RM 转换精灵 V 7.1 主程序脱壳`”一文后，自习了练习一下。做个笔记

