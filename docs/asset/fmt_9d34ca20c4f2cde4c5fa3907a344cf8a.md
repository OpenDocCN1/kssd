
### 0127B802 - 0127B819 代码分析

```
0127B802     E8 6DB9FFFF     call 01277174  
0127B807     8945 FC        mov dword ptr ss:[ebp-4],eax   
0127B80A     8B45 E0        mov eax,dword ptr ss:[ebp-20]  
0127B80D     8B00           mov eax,dword ptr ds:[eax]  
0127B80F     E8 C0E6FFFF    call 01279ED4  
0127B814     8BD0           mov edx,eax  
0127B816     0255 DF        add dl,byte ptr ss:[ebp-21]       ; 这里得到 dl  
0127B819     8B4D FC        mov ecx,dword ptr ss:[ebp-4]      ; [ebp-4C]就是我们要的函数  
0127B81C     8B45 F4        mov eax,dword ptr ss:[ebp-C]  
```

到了这里 `[ebp-4C]` 是我们需要的导入函数的地址，`dl` 中的值决定了是 `call(ff15)` 还是 `jmp(ff25)`

`dl` 中的值不同的程序是随机，找几个 `call 014F0000` 进去出来看一下就知道当前的程序中哪个对应 `ff15`，哪个对应 `ff25` 了

#### b 情况，进入 `call 0127A658`，一路 F8：

```
0127A7D6     8B7C82 68      mov edi,dword ptr ds:[edx+eax*4+68]  
0127A7DA     8B06           mov eax,dword ptr ds:[esi]  
0127A7DC     FFD7           call edi  
0127A7DE     8845 CA        mov byte ptr ss:[ebp-36],al  
0127A7E1     8B45 F4        mov eax,dword ptr ss:[ebp-C]  
0127A7E4     8A40 4A        mov al,byte ptr ds:[eax+4A]  
0127A7E7     3A45 EF        cmp al,byte ptr ss:[ebp-11]          ; al 的值决定是 FF15 还是 FF25  
0127A7EA     0F85 9C000000  jnz 0127A88C                         ; 不跳是 FF15 情况  
0127A7F0     EB 01          jmp short 0127A7F3  
```

`al` 中的值，不同程序是不同的。我的是 `F7`。

#### FF15：

```
0127A7F3     8B45 F4        mov eax,dword ptr ss:[ebp-C]  
0127A7F6     8B80 E0000000  mov eax,dword ptr ds:[eax+E0]  
0127A7FC     0145 FC        add dword ptr ss:[ebp-4],eax          ; [ebp-4]是解密出来的 API  
0127A7FF     EB 01          jmp short 0127A802                     ; 这里下断就能得到 API  
```

#### FF25：

```
0127A8A5     8B45 F4        mov eax,dword ptr ss:[ebp-C]  
0127A8A8     8B80 E0000000  mov eax,dword ptr ds:[eax+E0]  
0127A8AE     0145 FC        add dword ptr ss:[ebp-4],eax          ; [ebp-4]是解密出来的 API  
0127A8B1     8D45 0C        lea eax,dword ptr ss:[ebp+C]          ; 这里下断就能得到 API  
```

`0127A7FC` 或 `0127A8AE` 或 `0127B7E9` 执行完后 `[ebp-4]` 是需要的输入函数的地址，再看看 `[ebp-2c]`，如果它是 `FFFFFFFF`，说明这个导入函数调用是干净的，如果它有值，表示它的下一行也偷了。

因此只要在 `0127A7FF`、`0127A8B1`、`0127B819` 处下断即可获得 `API`。

### 3. Advanced Import Protection 的保护位置：

重新加载原程序，对 `004D5D9F`＋1（第一个 `E8` 我们不需要，要的是后面的 4 个字节）下内存写入断点：

```
004D5D9F     E8 5CA20101    call 014F0000  
```

第 2 次断下后停在这里：

```
0127BAD3     8945 00        mov dword ptr ss:[ebp],eax     // ebp 指向 004D5DA0，eax 写入后，变成 call 014F0000  
0127BAD6     6A 0A          push 0A  
0127BAD8     E8 7F9AFEFF    call 0126555C  
0127BADD     8BC8           mov ecx,eax  
0127BADF     038B E4000000  add ecx,dword ptr ds:[ebx+E4]  
0127BAE5     8BD6           mov edx,esi  
0127BAE7     8BC3           mov eax,ebx  
0127BAE9     E8 8EE5FFFF    call 0127A07C  
0127BAEE     FF0C24         dec dword ptr ss:[esp]  
0127BAF1     03B3 E4000000  add esi,dword ptr ds:[ebx+E4]  
0127BAF7     833C24 00      cmp dword ptr ss:[esp],0  
0127BAFB   ^ 0F87 55FEFFFF  ja 0127B956                   // 如果还有需要处理就跳上去  
0127BB01     53             push ebx  
0127BB02     E8 5D000000    call 0127BB64  
0127BB07     0183 EC000000  add dword ptr ds:[ebx+EC],eax  
0127BB0D     B0 01          mov al,1  
0127BB0F     83C4 24        add esp,24  
0127BB12     5D             pop ebp  
0127BB13     5F             pop edi  
0127BB14     5E             pop esi  
0127BB15     5B             pop ebx  
0127BB16     C3             retn  
```

跟随：`0127BAFB   ^ 0F87 55FEFFFF   ja 0127B956`

```
0127B956     33C0           xor eax,eax  
0127B958     8A07           mov al,byte ptr ds:[edi]  
0127B95A     8D0440         lea eax,dword ptr ds:[eax+eax*2]  
0127B95D     8B6C83 68      mov ebp,dword ptr ds:[ebx+eax*4+68]  
0127B961     8BC6           mov eax,esi  
0127B963     FFD5           call ebp                 
0127B965     8BE8           mov ebp,eax  
0127B967     036B 24        add ebp,dword ptr ds:[ebx+24]  
0127B96A     03AB E0000000  add ebp,dword ptr ds:[ebx+E0]  
0127B970     EB 01          jmp short 0127B973  
0127B973     33C0           xor eax,eax  
0127B975     8A47 09        mov al,byte ptr ds:[edi+9]  
0127B978     8D0440         lea eax,dword ptr ds:[eax+eax*2]  
0127B97B     8B5483 68      mov edx,dword ptr ds:[ebx+eax*4+68]  
0127B97F     8BC6           mov eax,esi  
0127B981     FFD2           call edx                   //call edx 结果在 eax  
0127B983     807B 20 00     cmp byte ptr ds:[ebx+20],0         //eax 可能是 1 或 0  
0127B987     0F85 3D010000  jnz 0127BACA  
0127B98D     3C 01          cmp al,1  
0127B98F     0F85 35010000  jnz 0127BACA  
0127B995     EB 01          jmp short 0127B998  
```

shoooo：

如果是 `1`，当前这个 `call 01200000` 处运行时，会重新回到调用地址，再进入导入函数

如果是 `0`，当前这个 `call 01200000` 进入导入函数后出来，不过这种方式比较邪恶，它可能做更多的事情

### 4. 获得所有 `call 014F0000` 的地址表：

这里想办法获得程序里“ `call    014F0000` ”语句所有的地址，上面代码中“`0127B981     FFD2    call edx` ”这个点比较好，除了能得到地址表，还能得知晓 `call 014F0000` 是按类型 `1` 来处理的，还是按类型 `2` 来处理的。

`OD` 载入，忽略除内存异常外的所有异常，`F9`，异常停住，`Ctrl＋G` 到 `0127B981  FFD2    call edx` 下断，取消异常忽略，`Shift＋F9` 直接断在这里：

```
0127B981     FFD2           call edx           // 断住，patch  
0127B983     807B 20 00     cmp byte ptr ds:[ebx+20],0    
0127B987     0F85 3D010000  jnz 0127BACA  
0127B98D     3C 01          cmp al,1  
0127B98F     0F85 35010000  jnz 0127BACA  
0127B995     EB 01          jmp short 0127B998  
```

堆栈：

```
0012FEF8   0000073C   // 有 73C 个加密  
0012FEFC   FE07987D  
0012FF00   0E3169DD  
0012FF04   00000000  
```

修改为：

```
0127B981   - E9 7A462B00     jmp 01530000          // 申请的空间 01530000  
0127B986     90             nop  
0127B987     0F85 3D010000   jnz 0127BACA  
0127B98D     3C 01          cmp al,1  
0127B98F     0F85 35010000   jnz 0127BACA  
0127B995     EB 01          jmp short 0127B998  
```

写上代码：

```
01530000     FFD2           call edx                  //壳原来的代码  
01530002     60             pushad  
01530003     8B1D 30005301  mov ebx,dword ptr ds:[1530030]    取缓存地址，设[1530030]=01530040，需要添入  
01530009     C1E0 1F        shl eax,1F  
0153000C     03C5           add eax,ebp  
0153000E     8903           mov dword ptr ds:[ebx],eax  
01530010     83C3 04        add ebx,4  
01530013     891D 30005301  mov dword ptr ds:[1530030],ebx  
01530019     61             popad  
0153001A     807B 20 00     cmp byte ptr ds:[ebx+20],0          //壳原来的代码  
0153001E   - E9 64B9D4FF    jmp 0127B987                      //跳转回去  
```

写入后数据窗口数据如下：

```
01530000  FF D2 60 8B 1D 30 00 53 01 C1 E0 1F 03 C5 89 03  襚?0.S 拎艍  
01530010  83 C3 04 89 1D 30 00 53 01 61 80 7B 20 00 E9 64  兠?0.Sa€{ .閐  
01530020  B9 D4 FF 00 00 00 00 00 00 00 00 00 00 00 00 00  乖.............  
01530030  40 00 53 01 00 00 00 00 00 00 00 00 00 00 00 00  @.S............  
01530040  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
```

其中红色部分需要额外添入，取得的地址就会放到这个地址开始的内存中。

`F4` 到 `0127BB01`：

```
0127BAF7     833C24 00      cmp dword ptr ss:[esp],0  
0127BAFB   ^ 0F87 55FEFFFF  ja 0127B956                // 如果还有需要处理就跳上去  
0127BB01     53             push ebx                // F4 到这里，处理完毕  
0127BB02     E8 5D000000    call 0127BB64  
```

查看数据窗口，此时得到所有 `API` 加密的地址：总共 `73C` 个

```
92 32 40 00 98 32 40 00 9E 32 40 00 A4 32 40 00 AA 32 40 00 B0 32 40 00 B6 32 40 00 BC 32 40 00  
C2 32 40 00 C8 32 40 00 CE 32 40 00 D4 32 40 00 DA 32 40 00 E0 32 40 00 E6 32 40 00 EC 32 40 00  
F2 32 40 00 F8 32 40 00 FE 32 40 00 04 33 40 00 0A 33 40 00 1C 33 40 00 22 33 40 00 28 33 40 00  
2E 33 40 00 34 33 40 00 3A 33 40 00 40 33 40 00 46 33 40 00 4C 33 40 00 6A 33 40 00 7C 33 40 00  
82 33 40 00 88 33 40 00 8E 33 40 00 94 33 40 00 9A 33 40 00 A6 33 40 00 BE 33 40 00 CA 33 40 00  
D6 33 40 00 DC 33 40 00 E2 33 40 00 E8 33 40 00 EE 33 40 00 F4 33 40 00 00 34 40 00 0C 34 40 00  
1E 34 40 00 24 34 40 00 2A 34 40 00 30 34 40 00 36 34 40 00 3C 34 40 00 42 34 40 00 48 34 40 00  
```
