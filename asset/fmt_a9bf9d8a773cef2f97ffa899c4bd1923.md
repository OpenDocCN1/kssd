
1 `00EC1420` `user32.dll` `018A` `IsChild`

1 `00EC1434` `wsock32.dll` `0010` `recv`

1 `00EC143C` `kernel32.dll` `01F9` `LockResource`

1 `00EC1440` `kernel32.dll` `00A8` `FindFirstFileA`

1 `00EC1444` `user32.dll` `0059` `CreateMenu`

1 `00EC1450` `kernel32.dll` `025B` `ResumeThread`

1 `00EC145C` `comctl32.dll` `002C` `ImageList_Create`

1 `00EC1460` `kernel32.dll` `00CE` `GetACP`

1 `00EC1488` `kernel32.dll` `012D` `GetFileType`

1 `00EC148C` `user32.dll` `019A` `KillTimer`

1 `00EC1490` `kernel32.dll` `0111` `GetCurrentThreadId`

1 `00EC1494` `kernel32.dll` `01B3` `GlobalUnlock`

1 `00EC149C` `advapi32.dll` `01A5` `RegOpenKeyExA`

1 `00EC14A0` `ws2_32.dll` `0008` `htonl`

1 `00EC14A4` `ntdll.dll` `0251` `RtlReAllocateHeap`

1 `00EC14A8` `advapi32.dll` `018C` `RegCloseKey`

1 `00EC14B0` `user32.dll` `01CF` `MoveWindow`

1 `00EC14B4` `advapi32.dll` `0195` `RegDeleteValueA`

1 `00EC14BC` `user32.dll` `005B` `CreateWindowExA`

1 `00EC14C0` `kernel32.dll` `0171` `GetStringTypeW`

1 `00EC14C4` `user32.dll` `003E` `CloseClipboard`

1 `00EC14CC` `advapi32.dll` `01AF` `RegQueryValueExA`

1 `00EC14D4` `user32.dll` `01A5` `LoadImageA`

1 `00EC14D8` `kernel32.dll` `00C6` `FreeEnvironmentStringsA`

1 `00EC14E0` `gdi32.dll` `0046` `CreatePen`

1 `00EC14E4` `kernel32.dll` `0158` `GetProcAddress`

1 `00EC14E8` `user32.dll` `01D9` `OpenClipboard`

1 `00EC14F0` `kernel32.dll` `016E` `GetStringTypeA`

1 `00EC14F4` `user32.dll` `0096` `DialogBoxParamA`

1 `00EC1504` `user32.dll` `010C` `GetForegroundWindow`

1 `00EC150C` `user32.dll` `0198` `IsZoomed`

1 `00EC1510` `kernel32.dll` `01BF` `HeapFree`

1 `00EC1514` `kernel32.dll` `012E` `GetFullPathNameA`

1 `00EC1518` `gdi32.dll` `017F` `GetTextMetricsA`

1 `00EC1528` `gdi32.dll` `0055` `DeleteObject`

1 `00EC152C` `kernel32.dll` `003A` `CreateFileMappingA`

1 `00EC1530` `user32.dll` `01F7` `RegisterClassA`

1 `00EC1534` `kernel32.dll` `002C` `CopyFileA`

1 `00EC1538` `user32.dll` `0101` `GetDC`

1 `00EC153C` `user32.dll` `0208` `ReleaseDC`

1 `00EC1540` `gdi32.dll` `01AE` `Polygon`

1 `00EC1544` `kernel32.dll` `0132` `GetLastError`

1 `00EC154C` `kernel32.dll` `02F5` `VirtualAlloc`

1 `00EC1554` `user32.dll` `000D` `BeginPaint`

1 `00EC155C` `ws2_32.dll` `0017` `socket`

1 `00EC1560` `kernel32.dll` `013F` `GetModuleHandleA`

1 `00EC1564` `gdi32.dll` `0192` `MoveToEx`

1 `00EC1568` `kernel32.dll` `01CC` `InitializeCriticalSection`

1 `00EC1578` `kernel32.dll` `02CA` `Sleep`

1 `00EC157C` `kernel32.dll` `013D` `GetModuleFileNameA`

1 `00EC1580` `shell32.dll` `0171` `ShellExecuteA`

1 `00EC1584` `gdi32.dll` `01D5` `SetBkColor`

从上面可以看出，每个 `dll` 对应的 `API` 在 `IAT` 中都不是连续的，而是乱插。这样很难用 `ImportREC` 完整还原 `IAT`，会把需要的函数也 `cut`。因此必须在程序入口处自己写代码来专门处理 `IAT`。思路是：把上述 `dll` 名和对应的 `API` 名移植到程序尾部，创建一个内存映射到 `012c1000` 处，然后根据 `dll` 名和 `API` 名用 `GetProcAddress` 得到所有函数入口地址，并写回其对应的地址。

`4dd466d0e1e0cc123afbe504da2462ca`
