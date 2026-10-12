
# 汇编指令分析

`10006A36`    `FF7424 38`       `push`    `dword ptr [esp+38]`  

`10006A3A`    `C2 3C00`         `retn`    `3C`                         ; VM 调用 API 的地方，下硬件执行  

# API 调用分析

然后分析他调用了哪些 API：

`0006F690`   `100096A0`  `/CALL` 到 `GetModuleHandleA`  

`0006F694`   `10002150`  `\pModule` = `"EXPLORER.EXE"`  

`0006F690`   `1000B6E2`  `/CALL` 到 `GetModuleHandleA` ；会连续调用两次，如果第一次就找到会执行真正目的！  

`0006F694`   `10002160`  `\pModule` = `"lqcyc52.cyc"`  

`0006F690`   `1000B6E2`  `/CALL` 到 `GetModuleHandleA` ；如果是第 2 次调用才成功，会用下面的方式搜索有多少杀毒软件  

`0006F694`   `10002160`  `\pModule` = `"lqcyc52.cyc"` ；因为我把 `lqcyc52.cyc` 改成了 `DLL`，所以要修改下这个数据  

`10002160`  `6C 71 63 79 63 35 32 2E 63 79 63`           `lqcyc52.cyc`  

`10002160`  `6C 71 63 79 63 35 32 2E 63 79 63 2E 64 6C 6C 00`  `lqcyc52.cyc.dll.`  

**后来发现下面一系列的 `GetModuleHandleA` 是判断是不是被这些进程载入了**

`0006F68C`   `1000B6A3`  `/CALL` 到 `GetModuleHandleA`  

`0006F690`   `10003038`  `\pModule` = `"dep360.exe"`  

`0006F68C`   `1000B6A3`  `/CALL` 到 `GetModuleHandleA`  

`0006F690`   `1000304C`  `\pModule` = `"360sd.exe"`  

`0006F68C`   `1000B6A3`  `/CALL` 到 `GetModuleHandleA`  

`0006F690`   `10003060`  `\pModule` = `"360rp.exe"`  

`0006F68C`   `1000B6A3`  `/CALL` 到 `GetModuleHandleA`  

`0006F690`   `10003074`  `\pModule` = `"360Safe.exe"`  

`0006F68C`   `1000B6A3`  `/CALL` 到 `GetModuleHandleA`  

`0006F690`   `10003088`  `\pModule` = `"DSMain.exe"`  

`0006F68C`   `1000B6A3`  `/CALL` 到 `GetModuleHandleA`  

`0006F690`   `1000309C`  `\pModule` = `"ZhuDongFangYu.exe"`  

`0006F68C`   `1000B6A3`  `/CALL` 到 `GetModuleHandleA`  

`0006F690`   `100030B0`  `\pModule` = `"360tray.exe"`  

`0006F68C`   `1000B6A3`  `/CALL` 到 `GetModuleHandleA`  

`0006F690`   `100030C4`  `\pModule` = `"360WDMain.exe"`  

`0006F68C`   `1000B6A3`  `/CALL` 到 `GetModuleHandleA`  

`0006F690`   `100030D8`  `\pModule` = `"360realpro.exe"`  

`0006F68C`   `1000B6A3`  `/CALL` 到 `GetModuleHandleA`  

`0006F690`   `100030EC`  `\pModule` = `"RsMain.exe"`  

`0006F68C`   `1000B6A3`  `/CALL` 到 `GetModuleHandleA`  

`0006F690`   `10003100`  `\pModule` = `"RsTray.exe"`  

`0006F68C`   `1000B6A3`  `/CALL` 到 `GetModuleHandleA`  

`0006F690`   `10003114`  `\pModule` = `"RavMonD.exe"`  

`0006F68C`   `1000B6A3`  `/CALL` 到 `GetModuleHandleA`  

`0006F690`   `10003128`  `\pModule` = `"kav32.exe"`  

`0006F68C`   `1000B6A3`  `/CALL` 到 `GetModuleHandleA`  

`0006F690`   `1000313C`  `\pModule` = `"kavstart.exe"`  

`0006F68C`   `1000B6A3`  `/CALL` 到 `GetModuleHandleA`  

`0006F690`   `10003150`  `\pModule` = `"liveupdate360.exe"`  

`0006F68C`   `1000B6A3`  `/CALL` 到 `GetModuleHandleA`  

`0006F690`   `10003164`  `\pModule` = `"egui.exe"`  

# 进程列表分析

看下到底要查多少进程：

`10003034`  `00 00 00 00 64 65 70 33 36 30 2E 65 78 65 00 00`  `....dep360.exe..`  

`10003044`  `00 00 00 00 00 00 00 00 33 36 30 73 64 2E 65 78`  `........360sd.ex`  

`10003054`  `65 00 00 00 00 00 00 00 00 00 00 00 33 36 30 72`  `e...........360r`  

`10003064`  `70 2E 65 78 65 00 00 00 00 00 00 00 00 00 00 00`  `p.exe...........`  

`10003074`  `33 36 30 53 61 66 65 2E 65 78 65 00 00 00 00 00`  `360Safe.exe.....`  

`10003084`  `00 00 00 00 44 53 4D 61 69 6E 2E 65 78 65 00 00`  `....DSMain.exe..`  

`10003094`  `00 00 00 00 00 00 00 00 5A 68 75 44 6F 6E 67 46`  `........ZhuDongF`  

`100030A4`  `61 6E 67 59 75 2E 65 78 65 00 00 00 33 36 30 74`  `angYu.exe...360t`  

`100030B4`  `72 61 79 2E 65 78 65 00 00 00 00 00 00 00 00 00`  `ray.exe.........`  

`100030C4`  `33 36 30 57 44 4D 61 69 6E 2E 65 78 65 00 00 00`  `360WDMain.exe...`  

`100030D4`  `00 00 00 00 33 36 30 72 65 61 6C 70 72 6F 2E 65`  `....360realpro.e`  

`100030E4`  `78 65 00 00 00 00 00 00 52 73 4D 61 69 6E 2E 65`  `xe......RsMain.e`  

`100030F4`  `78 65 00 00 00 00 00 00 00 00 00 00 52 73 54 72`  `xe..........RsTr`  

`10003104`  `61 79 2E 65 78 65 00 00 00 00 00 00 00 00 00 00`  `ay.exe..........`  

`10003114`  `52 61 76 4D 6F 6E 44 2E 65 78 65 00 00 00 00 00`  `RavMonD.exe.....`  

`10003124`  `00 00 00 00 6B 61 76 33 32 2E 65 78 65 00 00 00`  `....kav32.exe...`  

`10003134`  `00 00 00 00 00 00 00 00 6B 61 76 73 74 61 72 74`  `........kavstart`  

`10003144`  `2E 65 78 65 00 00 00 00 00 00 00 00 6C 69 76 65`  `.exe........live`  

`10003154`  `75 70 64 61 74 65 33 36 30 2E 65 78 65 00 00 00`  `update360.exe...`  

`10003164`  `65 67 75 69 2E 65 78 65 00 00 00 00 00 00 00 00`  `egui.exe........`  

`10003174`  `00 00 00 00 65 6B 72 6E 2E 65 78 65 00 00 00 00`  `....ekrn.exe....`  

`10003184`  `00 00 00 00 00 00 00 00 6F 6C 6C 79 64 62 67 2E`  `........ollydbg.`  

`10003194`  `65 78 65 00 00 00 00 00 00 00 00 00 4F 44 62 67`  `exe.........ODbg`  

`100031A4`  `53 63 72 69 70 74 2E 64 6C 6C 00 00 00 00 00 00`  `Script.dll......`  

`100031B4`  `4F 6C 6C 79 44 75 6D 70 2E 64 6C 6C 00 00 00 00`  `OllyDump.dll....`  

`100031C4`  `00 00 00 00 49 63 65 53 77 6F 72 64 2E 65 78 65`  `....IceSword.exe`  

`100031D4`  `00 00 00 00 00 00 00 00 77 73 79 73 63 68 65 63`  `........wsyschec`  

`100031E4`  `6B 2E 65 78 65 00 00 00 00 00 00 00 53 6E 69 70`  `k.exe.......Snip`  

`100031F4`  `65 53 77 6F 72 64 2E 65 78 65 00 00 00 00 00 00`  `eSword.exe......`  

`10003204`  `61 76 70 2E 65 78 65 00 00 00 00 00 00 00 00 00`  `avp.exe.........`  

`10003214`  `00 00 00 00 57 53 6F 63 6B 45 78 70 65 72 74 2E`  `....WSockExpert.`  

`10003224`  `65 78 65 00 00 00 00 00 57 53 6F 63 6B 48 6F 6F`  `exe.....WSockHoo`  

`10003234`  `6B 2E 64 6C 6C 00 00 00 00 00 00 00 57 70 65 53`  `k.dll.......WpeS`  

`10003244`  `70 79 2E 64 6C 6C 00 00 00 00 00 00 00 00 00 00`  `py.dll..........`  

`10003254`  `6B 70 70 74 72 61 79 2E 65 78 65 00 00 00 00 00`  `kpptray.exe.....`  

`10003264`  `00 00 00 00 6B 70 70 6D 61 69 6E 2E 65 78 65 00`  `....kppmain.exe.`  

`10003274`  `00 00 00 00 00 00 00 00 30 30 30 30 6B 73 64 65`  `........0000ksde`  

`10003284`  `73 6B 2E 65 78 65 00 00 00 00 00 00 6B 70 70 73`  `sk.exe......kpps`  

`10003294`  `65 72 76 2E 65 78 65 00 00 00 00 00 00 00 00 00`  `erv.exe.........`  

；＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝这里是真正目的

# 核心逻辑分析

`0006F274`   `100095E9`  `/CALL` 到 `GetSystemDirectoryA`  

`0006F278`   `100036E0`  `|Buffer` = `lqcyc52\_.100036E0`  

`0006F27C`   `00000103`  `\BufSize` = `103` (259.)  

取出来后和 `"\\usp10.dll"` 连接

`100036E0`  `43 3A 5C 57 49 4E 44 4F 57 53 5C 73 79 73 74 65`  `C:\WINDOWS\syste`  

`100036F0`  `6D 33 32 5C 75 73 70 31 30 2E 64 6C 6C 00 00 00`  `m32\usp10.dll...`  

`0006F274`   `10008A2F`  `/CALL` 到 `DeleteFileA` ；删除。。。  

`0006F278`   `0006F488`  `\FileName` = `"C:\WINDOWS\system32\dllcache\usp10.dll"`  

`0006F270`   `1000892B`  `/CALL` 到 `MoveFileA` ；这之后出现文件保护提示  

`0006F274`   `100036E0`  `|ExistingName` = `"C:\WINDOWS\system32\usp10.dll"`  

`0006F278`   `0006F384`  `\NewName` = `"C:\WINDOWS\system32\cybkus10.dll"`  

`0006F26C`   `10009072`  `/CALL` 到 `CopyFileA` ；用 `lqcyc52.cyc` 代替 `usp10.dll`  

`0006F270`   `0006F280`  `|ExistingFileName` = `"C:\WINDOWS\system32\lqcyc52.cyc"`  

`0006F274`   `0006F488`  `|NewFileName` = `"C:\WINDOWS\system32\dllcache\usp10.dll"`  

`0006F278`   `00000001`  `\FailIfExists` = `TRUE`  

`0006F26C`   `10007DED`  `/CALL` 到 `CopyFileA`  

`0006F270`   `0006F280`  `|ExistingFileName` = `"C:\WINDOWS\system32\lqcyc52.cyc"`  

`0006F274`   `100036E0`  `|NewFileName` = `"C:\WINDOWS\system32\usp10.dll"`  

`0006F278`   `00000001`  `\FailIfExists` = `TRUE`  

`0006F274`   `10008238`  `/CALL` 到 `PathFileExistsA`  

`0006F278`   `0006F384`  `\Path` = `"C:\WINDOWS\system32\cybkus10.dll"`  

`0006F270`   `100096A5`  `/CALL` 到 `GetModuleHandleA`  

`0006F274`   `00000000`  `\pModule` = `NULL`  

`0006F270`   `1000B0E1`  `/CALL` 到 `GetModuleFileNameA` ；这里应该是想查看是谁调用的这个 `DLL`，保险其间我们把他改一下  

`0006F274`   `00400000`  `|hModule` = `00400000` (`LOADDLL`)  

`0006F278`   `0006F58C`  `|PathBuffer` = `0006F58C`  

`0006F27C`   `00000103`  `\BufSize` = `103` (259.)  

`0006F58C`  `45 3A 5C B9 A4 BE DF CF E4 5C 4F 6C 6C 79 49 43`  `E:\工具箱\OllyIC`  

`0006F59C`  `45 5C 4C 4F 41 44 44 4C 4C 2E 45 58 45`          `E\LOADDLL.EXE`  

改成：

`0006F58C`  `43 3A 5C 57 49 4E 44 4F 57 53 5C 73 79 73 74 65`  `C:\WINDOWS\syste`  

`0006F59C`  `6D 33 32 5C 74 61 73 6B 6E 67 72 2E 65 78 65 00`  `m32\taskngr.exe.`  

`0006F258`   `10008F6E`  `/CALL` 到 `RegOpenKeyExA`  

`0006F25C`   `80000002`  `|hKey` = `HKEY_LOCAL_MACHINE`  

`0006F260`   `10002080`  `|Subkey` = `"Software\360Safe\menuext\LiveUpdate360"`  

`0006F264`   `00000000`  `|Reserved` = `0`  

`0006F268`   `00020019`  `|Access` = `KEY_READ`  

`0006F26C`   `0006F270`  `\pHandle` = `0006F270`  

`0006F254`   `10008059`  `/CALL` 到 `RegQueryValueExA`  

`0006F258`   `00000000`  `|hKey` = `0`  

`0006F25C`   `100020A8`  `|ValueName` = `"Application"`  

`0006F260`   `00000000`  `|Reserved` = `NULL`  

`0006F264`   `0006F278`  `|pValueType` = `0006F278`
