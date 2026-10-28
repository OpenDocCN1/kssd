
```
004A65F5 <>|> \B>mov     ecx, 312                                      ;  从这里我们可以看到 WM_HOTKEY 312

004A65FA   |.  3>cmp     eax, ecx

004A65FC   |.  7>jnz     short <MalwareD.loc_4A6617>

............

004A660D   |.  E>call    MalwareD.0048F684                     ;这里肯定就是处理流程了

004A6612   |.^ E>jmp     <MalwareD.loc_4A63DB>

/////////////////////////////

0048F684   /$  5>push    esi                                           ; 

0048F685   |.  5>push    edi

0048F686   |.  3>xor     edi, edi

0048F688   |.  8>mov     esi, ecx

0048F68A   |.  3>cmp     dword ptr ds:[<dword_5D4D58>], edi

0048F690   |.  0>jnz     <MalwareD.loc_48F7EF>

0048F696   |.  8>mov     eax, dword ptr ss:[esp+10]        ;这里将热键的ID传入eax,然后进行查找匹配的分支.

0048F69A       3>xor     ecx, ecx

0048F69C       4>inc     ecx

0048F69D       3>cmp     eax, ecx

0048F69F   |.  7>jnz     short <MalwareD.loc_48F6AD>

0048F6A1   |.  8>mov     ecx, esi

0048F6A3   |.  E>call    <MalwareD._ShowMainWindow>

0048F6A8   |.  E>jmp     <MalwareD.loc_48F7EF>

0048F6AD <>|>  3>cmp     dword ptr ds:[<_IsLocking***>], edi           ;  loc_48F6AD

0048F6B3   |.  0>jnz     <MalwareD.loc_48F7EF>

0048F6B9   |.  6>push    2

0048F6BB   |.  5>pop     edx
```

小结：通过动态调试，我们可以知道程序是根据`WM_HotKey`的`ID`来进行分支匹配的。在`0048F69A`跳到我们的代码进行分支调用。

至此，所需的所有必备条件已具备。下面开始进行`patch`代码和找空间写我们的代码。

原以为很容易就能找到大片的空隙来写代码了，但经过测试发现，程序的`.text`段的空隙很小（为了这个问题可是PE头搞个差不多明白了：））

![](https://bbs.kanxue.com/upload/attach/201011/170_ngk5nlcfmr50dyk.png)
![](https://bbs.kanxue.com/upload/attach/201011/170_yfefjsnkx41dic1.png)

没办法，只好增加区段了。正好利用所学，手动增加区段：

整理一下思路：

首先需要构建并增加一个段头，然后修改文件的`NumberOfSections`、`SizeofImages`，根据lordPE的区段数据，我们构建的新区段头如下：

```
typedef struct _IMAGE_SECTION_HEADER {

    BYTE    Name[IMAGE_SIZEOF_SHORT_NAME];     ".FSW"

    union {

            DWORD   PhysicalAddress;                    00000800h //我们需要的大小

            DWORD   VirtualSize;

    } Misc;

    DWORD   VirtualAddress;		00264000h  // .rsrc 段的Voffset+Vsize后1000对齐

    DWORD   SizeOfRawData;		00000800h	// 磁盘文件对齐大小.

    DWORD   PointerToRawData;		0024F200h	//  .rsrc 段的Roffset+RSize后200对齐

    DWORD   PointerToRelocations;		00000000h

    DWORD   PointerToLinenumbers;	00000000h	

    WORD    NumberOfRelocations;		0000h	

    WORD    NumberOfLinenumbers;		0000h	

    DWORD   Characteristics;		E0000020h // 可读可写可执行+代码.

} IMAGE_SECTION_HEADER, *PIMAGE_SECTION_HEADER;
```

将以上数值展开后如下：

```
2E465357 00000000 00080000 00402600 00080000 00F22400 00000000  00000000  0000  0000  200000E0
```

用十六进制工具将区段头写入`exe`文件

![](https://bbs.kanxue.com/upload/attach/201011/170_0nxrc1uv6xr1j18.png)

然后修改

- `NumberOfSections`：4->5
- `SizeofImages`：`(00264000h  +800h) 1000对齐=265000h`

![](https://bbs.kanxue.com/upload/attach/201011/170_m08l6q3wccil5gm.png)

然后用16进制编辑器在文件的`24f200`后插入`800h`的数据块（插入数据时，发现这里竟然有数据，不知什么原因，请大牛指点）。

最后的效果如图：

![](https://bbs.kanxue.com/upload/attach/201011/170_x097ymx6q4q28yr.png)

好了，有足够的空间了，下面我们就开始写代码了。

需要Patch3个地方

- 响应热键的地方
- 反注册热键的地方
- 注册热键的地方

先规划一下我们空间的使用：

由于注册热键时，需要用到`ID`、`fsModifiers`、`VK`，其中`ID`是一个`DWORD`变量，用来调用`UnRegisterHotKey`。

`fsModifiers`、`VK`这两个合并成一个`DWORD`，用来注册热键（如 `CTRL+SHIFT+ALT+1` 就是 `00070031`）

我们要注册的6个热键转换过来就是

```
00070031
00070032
00070033
00070034
00070035
00070036
```

这6个`DWORD`数据加上6个`DOWRD`变量共需要48个字节的空间。这些数据就放在我们新开辟空间的起始位置：`VA :664000` 后面再跟6个`DWORD`的变量。

先把这几个热键码写到程序中（十六进制编辑器搞定），注意字节序。

//////

这样我们就把代码位置放在`664000h+40h`的地方。

开始处理"响应热键"的地方

```
0048F684   /$ >push    esi

0048F685   |. >push    edi

0048F686   |. >xor     edi, edi

0048F688   |. >mov     esi, ecx

0048F68A   |. >cmp     dword ptr ds:[5D4D58], edi

0048F690   |. >jnz     MalwareD.0048F7EF

0048F696   |. >mov     eax, dword ptr ss:[esp+10]

0048F69A      >xor     ecx, ecx                                         // 改成 jmp     MalwareD.00664040

0048F69C      >inc     ecx

0048F69D     >cmp     eax, ecx                                       // 会占用到这里,这三行代码将在我们的代码中实现

0048F69F   |. >jnz     short MalwareD.0048F6AD

0048F6A1   |. >mov     ecx, esi
```

在`00664040`处写代码处理相应的热键消息

```
00589D80      >pushad

00589D81      >cmp     dword ptr ds:[5D4A5C], edi

00589D87      >jnz     short MalwareD.00589DA1

00589D89      >cmp     dword ptr ds:[5D4A60], edi

00589D8F      >jnz     short MalwareD.00589DA1

00589D91      >cmp     dword ptr ds:[5D4A64], edi

00589D97      >jnz     short MalwareD.00589DA1

00589D99      >cmp     dword ptr ds:[5D4A68], edi

00589D9F      >je      short MalwareD.00589DFE

00589DA1      >cmp     eax, 101

00589DA6      >jnz     short MalwareD.00589DAF

00589DA8      >call    MalwareD.0049079B

00589DAD      >jmp     short MalwareD.00589DF5

00589DAF      >cmp     eax, 102

00589DB4      >jnz     short MalwareD.00589DBD

00589DB6      >call    MalwareD.004907A7

00589DBB      >jmp     short MalwareD.00589DF5

00589DBD      >cmp     eax, 103

00589DC2      >jnz     short MalwareD.00589DCB

00589DC4      >call    MalwareD.004907B3

00589DC9      >jmp     short MalwareD.00589DF5

00589DCB      >cmp     eax, 104

00589DD0      >jnz     short MalwareD.00589DD9

00589DD2      >call    MalwareD.004907BF

00589DD7      >jmp     short MalwareD.00589DF5

00589DD9      >cmp     eax, 105

00589DDE      >jnz     short MalwareD.00589DE7

00589DE0      >call    MalwareD.004907CB

00589DE5      >jmp     short MalwareD.00589DF5

00589DE7      >cmp     eax, 106

00589DEC      >jnz     short MalwareD.00589DFE

00589DEE      >call    MalwareD.004907D7

00589DF3      >jmp     short MalwareD.00589DF5

00589DF5      >sub     esp, 10                                         // 注意 这里,完全是为是平衡栈

00589DF8      >popad

00589DF9     ^>jmp     MalwareD.0048F7EF

00589DFE      >popad

00589DFF      >xor     ecx, ecx

00589E01      >inc     ecx

00589E02      >cmp     eax, ecx                                      //00589DFF-00589E02是实现原始代码.

00589E04     ^>jmp     MalwareD.0048F69F
```

////////////////////////// OK响应热键的消息处理功能添加完成.

下面开始反注册热键：

```
0046294E   |. >push    MalwareD.005D4BB4

00462953   |. >mov     ecx, esi

00462955   |. >call    MalwareD.004628DD

0046295A   |. >push    MalwareD.005D4BB8

0046295F      >mov     ecx, esi		// 改 jmp     MalwareD.006640CD

00462961      >call    MalwareD.004628DD	// 填充 2个 nop 同样,把这些原始代码搬到我们的空间中.	 

00462966   |. >pop     esi
```

在`.006640CD`处写代码反注册热键

```
006640CD     8>mov     ecx, esi

006640CF     E>call    MalwareD.004628DD

006640D4     6>push    MalwareD.00664018

006640D9     8>mov     ecx, esi

006640DB     E>call    MalwareD.004628DD

006640E0     6>push    MalwareD.0066401C

006640E5     8>mov     ecx, esi

006640E7     E>call    MalwareD.004628DD

006640EC     6>push    MalwareD.00664020

006640F1     8>mov     ecx, esi

006640F3     E>call    MalwareD.004628DD

006640F8     6>push    MalwareD.00664024

006640FD     8>mov     ecx, esi

006640FF     E>call    MalwareD.004628DD

00664104     6>push    MalwareD.00664028

00664109     8>mov     ecx, esi

0066410B     E>call    MalwareD.004628DD

00664110     6>push    MalwareD.0066402C

00664115     8>mov     ecx, esi

00664117     E>call    MalwareD.004628DD

0066411C   - E>jmp     MalwareD.00462966
```

////////////////////////// OK反注册热键功能添加完成.

下面开始注册热键：

```
0047BF87   |. >push    9

0047BF89   |. >push    MalwareD.005D4BB8

0047BF8E   |. >call    MalwareD.0047BDD8

0047BF93      >mov     edi, dword ptr ss:[ebp-10] // 改  jmp     MalwareD.00664125

0047BF96      >cmp     dword ptr ds:[edi-8], 0      // 填充 2个 nop 同样,把这些原始代码搬到我们的空间中.	 		

0047BF9A   |. >je      short MalwareD.0047BFE6

0047BF9C   |. >mov     eax, dword ptr ds:[5C9084]
```

在`.00664125`处写代码注册热键
