
# 修改游戏进度的保存路径（非菜鸟可以直接略过）

> 原文：[`www.pediy.com/kssd/pediy10/75349.html`](https://www.pediy.com/kssd/pediy10/75349.html)

**【文章标题】**: 修改游戏进度的保存路径

**【文章作者】**: `windale`

**【下载地址】**: 游戏运行所需文件过大，不提供下载

**【作者声明】**: 本人菜鸟一个，此文章没有技术含量，只是叙述了一下思路。希望对其他菜鸟和新手有所帮助！

---

### 【详细过程】

`World Of Goo` 是一款独具创意的益智游戏，并且画面和音乐的制作都相当精致。由于我安装了影子系统，电脑重启之后游戏的进度就丢失了。我使用的是单一影子模式，仅对系统盘执行保护，所以断定游戏的进度肯定是存放在系统盘中。按照常规，相关文件应该会保存在 `C:\Documents and Settings\All Users\Application Data` 里面。结果不出所料，`C:\Documents and Settings\All Users\Application Data\2DBoy\WorldOfGoo` 里面有个 `pers2.dat` 文件，应该就是游戏进度数据了。如果能将文件保存的路径改在非系统盘下，就不会丢失进度了。不知道会不会很难，先试试再说！

于是先用 `PEiD` 查壳，结果没有加壳，省去了脱壳的麻烦，编译器是 `Microsoft Visual Studio .NET 8.0`。

然后用 `OD` 载入。先把 `pers2.dat` 文件给删掉，程序发现文件不存在就会创建一个新的。输入命令 `bp CreateFileA` 下断。可是中断之后，程序始终在系统领空，跳不出来。于是考虑别的办法。因为文件名 `pers2.dat` 应该是固定的，所以在程序中应该会出现相应的字符串。使用插件 `Ultra String Reference` -> `Find ASCII`，之后在结果中查找字符串 `"pers2.dat"`，果然找到了。而且在其上方依次有字符串 `"\\2dboy"`、`"\\worldofgoo"` 和 `"\\"`，看来此处附近就是执行文件操作的代码。双击其中的一个，来到了这个位置：

```
004B3880  |.  8D5424 18     lea     edx, dword ptr [esp+18]
004B3884  |.  52            push    edx
004B3885  |.  6A 00         push    0
004B3887  |.  6A 00         push    0
004B3889  |.  6A 23         push    23
004B388B  |.  6A 00         push    0
004B388D  |.  894F 04       mov     dword ptr [edi+4], ecx
004B3890  |.  FF15 14D55800 call    dword ptr [<&SHELL32.SHGetFolder>;  SHELL32.SHGetFolderPathA
004B3896  |.  8D4424 18     lea     eax, dword ptr [esp+18]
004B389A  |.  50            push    eax
004B389B  |.  8BCE          mov     ecx, esi
004B389D  |.  FF15 5CD25800 call    dword ptr [<&MSVCP80.std::basic_>;  MSVCP80.std::basic_string<char,std::char_traits<char>,std::allocator<char> >::append
004B38A3  |.  68 F4295E00   push    005E29F4                          ;  ASCII "\\2DBoy"
004B38A8  |.  8BCE          mov     ecx, esi
004B38AA  |.  FF15 5CD25800 call    dword ptr [<&MSVCP80.std::basic_>;  MSVCP80.std::basic_string<char,std::char_traits<char>,std::allocator<char> >::append
004B38B0  |.  837E 18 10    cmp     dword ptr [esi+18], 10
004B38B4  |.  72 05         jb      short 004B38BB
004B38B6  |.  8B46 04       mov     eax, dword ptr [esi+4]
004B38B9  |.  EB 03         jmp     short 004B38BE
004B38BB  |>  8D46 04       lea     eax, dword ptr [esi+4]
004B38BE  |>  8B1D 24D15800 mov     ebx, dword ptr [<&KERNEL32.Creat>;  kernel32.CreateDirectoryA
004B38C4  |.  6A 00         push    0                                 ; /pSecurity = NULL
004B38C6  |.  50            push    eax                               ; |Path
004B38C7  |.  FFD3          call    ebx                               ; \CreateDirectoryA
004B38C9  |.  8B2D 14D15800 mov     ebp, dword ptr [<&KERNEL32.GetLa>;  ntdll.RtlGetLastWin32Error
004B38CF  |.  FFD5          call    ebp                               ; [GetLastError
004B38D1  |.  68 FC295E00   push    005E29FC                          ;  ASCII "\\WorldOfGoo"
004B38D6  |.  8BCE          mov     ecx, esi
004B38D8  |.  FF15 5CD25800 call    dword ptr [<&MSVCP80.std::basic_>;  MSVCP80.std::basic_string<char,std::char_traits<char>,std::allocator<char> >::append
004B38DE  |.  837E 18 10    cmp     dword ptr [esi+18], 10
004B38E2  |.  72 05         jb      short 004B38E9
004B38E4  |.  8B46 04       mov     eax, dword ptr [esi+4]
004B38E7  |.  EB 03         jmp     short 004B38EC
004B38E9  |>  8D46 04       lea     eax, dword ptr [esi+4]
004B38EC  |>  6A 00         push    0
004B38EE  |.  50            push    eax
004B38EF  |.  FFD3          call    ebx
004B38F1  |.  FFD5          call    ebp
004B38F3  |.  68 082A5E00   push    005E2A08
004B38F8  |.  8BCE          mov     ecx, esi
004B38FA  |.  FF15 5CD25800 call    dword ptr [<&MSVCP80.std::basic_>;  MSVCP80.std::basic_string<char,std::char_traits<char>,std::allocator<char> >::append
004B3900  |.  68 0C2A5E00   push    005E2A0C                          ;  pers2.dat
004B3905  |.  8BCE          mov     ecx, esi
004B3907  |.  FF15 5CD25800 call    dword ptr [<&MSVCP80.std::basic_>;  MSVCP80.std::basic_string<char,std::char_traits<char>,std::allocator<char> >::append
004B390D  |.  57            push    edi                               ; /Arg1
004B390E  |.  E8 7D020000   call    004B3B90                          ; \WorldOfG.004B3B90
```

可以看到，`004B3890` 处调用了 `SHGetFolderPathA` 函数，十分可疑。于是在此处下断，运行程序，在此处步过，这时看到函数返回的路径是 `"C:\Documents and Settings\All Users\Application Data"`。下面就不用再分析了，此处已经八九不离十了。

这个函数有点陌生，查了一下 `MSDN`，得到其原型如下：

```
HRESULT SHGetFolderPath(
  HWND hwndOwner,
  int nFolder,
  HANDLE hToken,
  DWORD dwFlags,
  LPTSTR pszPath
);
```

其中第二个参数的解释如下：

**引用**: `nFolder`

`[in]` A `CSIDL` value that identifies the folder whose path is to be retrieved. Only real folders are valid. If a virtual folder is specified, this function fails. You can force creation of a folder by combining the folder's `CSIDL` with `CSIDL_FLAG_CREATE`.

也就是说，`nFolder` 的值由 `CSIDL` 指定，标识了要检索的文件夹。

然后又在 `MSDN` 中找到了对 `CSIDL` 的描述，列举了各种文件夹所对应的常量，其中有个 `CSIDL_PERSONAL`，代表“我的文档”文件夹。正好我把“我的文档”移到了 `D` 盘，不受影子系统的保护，那么把 `nFolder` 参数改成 `CSIDL_PERSONAL` 应该就行了。

查看原程序的代码，此参数的值是 `23`，那么应该改成多少呢？正好电脑上安装了 `VC++ 6.0`，在其安装目录搜索了一下，结果在 `SHLOBJ.H` 文件中找到了相关定义：

```
#define CSIDL_PERSONAL            0x0005
```

马上用 `WinHex` 打开游戏主程序，只有这一处调用 `SHGetFolderPathA` 函数，将此处的 `23` 修改为 `05`，保存。

重新运行游戏，果然在“我的文档”中创建了进度数据文件。重启电脑，运行游戏，进度没有丢失。大功告成！
