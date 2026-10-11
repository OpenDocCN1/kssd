
# [Armadillo][Import Table Elimination+Code Splicing]WebCrypt Pro 2000 v5

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-575.htm`](https://www.pediy.com/kssd/pediy08/pediy8-575.htm)

`Armadillo 1.xx - 2.xx -> Silicon Realms Toolworks [Overlay]`

单进程

`ArmadilloFindProtected`

`<Protection Options>`

- Standard protection or Minimum protection
- Enable Import Table Elimination
- Enable Strategic Code Splicing

`OD`载入：

```
004E71DC we>  55               push ebp           ; 入口点
004E71DD       8BEC            mov ebp,esp
004E71DF       6A FF           push -1
```

`bp GetModuleHandleA+7`

```
00128D08  /0012CF70
00128D0C  |00B805CD    返回到 00B805CD 来自 KERNEL32.GetModuleHandleA
00128D10  |00B92D60    ASCII "kernel32.dll"
00128D14  |00B93F60    ASCII "VirtualAlloc"
00128D18  |00000001

00128D08  /0012CF70
00128D0C  |00B805EA    返回到 00B805EA 来自 KERNEL32.GetModuleHandleA
00128D10  |00B92D60    ASCII "kernel32.dll"
00128D14  |00B93F54    ASCII "VirtualFree"
00128D18  |00000001

001080F4  /00108130
001080F8  |00B89A41    返回到 00B89A41 来自 KERNEL32.GetModuleHandleA
001080FC  |00000000

...中间省略一些，因为我们要见的是 kernel32.dll

00128A80  /00128D0C
00128A84  |00B69900    返回到 00B69900 来自 KERNEL32.GetModuleHandleA
00128A88  |00128BC0    ASCII "kernel32.dll"
```

可以返回：

```
00B69900      8B0D 3881B900   mov ecx,dword ptr ds:[B98138]
00B69906      89040E          mov dword ptr ds:[esi+ecx],eax
00B69909      A1 3881B900     mov eax,dword ptr ds:[B98138]
00B6990E      393C06          cmp dword ptr ds:[esi+eax],edi
00B69911      75 16           jnz short 00B69929
00B69913      8D85 B4FEFFFF   lea eax,dword ptr ss:[ebp-14C]
00B69919      50              push eax
00B6991A      FF15 D4D0B800   call dword ptr ds:[B8D0D4]       ; KERNEL32.LoadLibraryA
00B69920      8B0D 3881B900   mov ecx,dword ptr ds:[B98138]
00B69926      89040E          mov dword ptr ds:[esi+ecx],eax
00B69929      A1 3881B900     mov eax,dword ptr ds:[B98138]
00B6992E      393C06          cmp dword ptr ds:[esi+eax],edi
00B69931     /0F84 AD000000   je 00B699E4                       ; magic jump
00B699E4      83C3 0C         add ebx,0C
00B699E7      83C6 04         add esi,4
00B699EA      397B FC         cmp dword ptr ds:[ebx-4],edi
00B699ED    ^ 0F85 CCFEFFFF   jnz 00B698BF
00B699F3     /EB 03           jmp short 00B699F8                ; 断下后取消修改
00B699F5      D6              salc
```

有些不用撤销修改，有些需要，要不就无法调试。

`bp GetCurrentThreadId`

```
0012D798   00B88A24    /CALL 到 GetCurrentThreadId 来自 00B88A1E
0012D79C   0012FF2C
```

返回后 `F8` 步进到：

```
00B88AB7      FFD7          call edi                          ; webpro5.004AE268
004AE268      55            push ebp                          ; OEP
004AE269      8BEC          mov ebp,esp
004AE26B      83C4 F0       add esp,-10
004AE26E      53            push ebx
```

`RVA` 定位：

跟随 `call`，找 `jmp`：

```
00406224   - FF25 8CC0BB00   jmp dword ptr ds:[BBC08C]        ; KERNEL32.GetModuleHandleA
0040622A      8BC0          mov eax,eax
0040622C   - FF25 94C3BB00   jmp dword ptr ds:[BBC394]        ; KERNEL32.LocalAlloc
00406232      8BC0          mov eax,eax
00406234   - FF25 D4C5BB00   jmp dword ptr ds:[BBC5D4]        ; KERNEL32.TlsGetValue
```

`dd BBC08C:`

```
00BBBFE0  00000000
00BBBFE4  00000000
00BBBFE8  000B0133
00BBBFEC  00180700
00BBBFF0  77E06D86  USER32.CreatePopupMenu
00BBBFF4  77E6BF41  KERNEL32.ResetEvent
00BBBFF8  77E0757C  USER32.CreateMenu
00BBBFFC  77E6AB06  KERNEL32.GetModuleHandleA
00BBC000  779B92D0  OLEAUT32.SysReAllocStringLen
00BBC004  77E1D86D  USER32.CreateIcon
00BBC008  77E7F362  KERNEL32.GetExitCodeThread
...
00BBC0A0  77F46046  GDI32.SetMapMode
00BBC0A4  77DF80D8  USER32.ShowCursor
00BBC0A8  00B6A8F3
00BBC0AC  77E0A1DD  USER32.SetClipboardData
00BBC0B0  77E7AC6E  KERNEL32.Sleep
00BBC0B4  77F6E259  GDI32.GetEnhMetaFileDescriptionA
...
00BBC94C  77DF383C  USER32.PtInRect
00BBC950  77E7EB3A  KERNEL32.GetVersion
00BBC954  00B6A962
00BBC958  796D1E76  ADVAPI32.RegCloseKey
00BBC95C  00B6A957
00BBC960  00B6A927
00BBC964  77DF63BD  USER32.SetForegroundWindow
00BBC968  BAADF00D
00BBC96C  BAADF00D
00BBC970  ABABABAB
00BBC974  ABABABAB
00BBC978  00000000
00BBC97C  00000000
```

`RVA=00BBBFE4  SIZE=994`

`code splicing` 定位：

从 `OEP` 处往下搜索：`E9`

```
004013A8   - E9 53EC3703     jmp 03780000        // 这个就是 code splicing 的地址
```

`ArmInline`：

上 `ArmInline` 修复 `code splicing`：

`OD` 附加可以看到 `id`：

![](img/f2d76a289cc4e440c7a32692b0fe0ebf.jpg)

`Alt+M` 查看 `code splicing` 数据:

![](img/8d3fe48afec72cd394fa54cb8ac55fea.jpg)

![](img/05cf74f063323b9dd85c852ec49b2585.jpg)

![](img/a86391b27f6b222447926789b46bb315.jpg)

修复后就可以 `dump` 了。

接下来修复 `IAT` 乱序：

在 `idata` 找空间：

![](img/d132d7263071d18f762ecfe9b9dfaa8a.jpg)

搜索 `kernel32.dll`：

![](img/ac6f63937da60ad27a29d8531739b683.jpg)

放在 `004B3D88` 处。

![](img/6b15c2c84330847d8d4982fd50dc3998.jpg)

上 `ImportREC`：`OEP=000AE268`，自动搜索，没有无效函数，修复。

运行 `ok`，`Borland Delphi 6.0 - 7.0`。

脱壳后不再提示注册，应该没有功能限制。

`4dd466d0e1e0cc123afbe504da2462ca`
