

# 输入表重建工具 ImportREC 

> 原文：[`www.pediy.com/kssd/tutorial/chap8-5-41.htm`](https://www.pediy.com/kssd/tutorial/chap8-5-41.htm)

　　 Import REConstructor 可以从杂乱的 IAT 中重建一个新的 Import 表（例如加壳软件等），它可以重建 Import 表的描述符、IAT 和所有的 ASCII 函数名。用它配合手动脱壳，可以脱 UPX、CDilla1、PECompact、PKLite32、Shrinker、ASPack, ASProtect 等壳。该工具位于：光盘\\tools\\PE tools\\Rebuilders\\Import REConstructor。  
在运行 Import REConstructor 之前，必须满足如下条件：  
1） 目标文件己完全被 Dump 到另一文件；  
2） 目标文件必须正在运行中；  
3） 事先要找到真正的入口点（OEP）；  
4） 最好加载 IceDump，这样建立的输入表较少存在跨平台的问题。

步骤如下：

（1）找被脱壳的入口点（OEP）；  
（2）完全 Dump 目标文件；  
（3）运行 Import REConstructor 和需要脱壳的应用程序；  
（4）在 Import REConstructor 下拉列表框中选择应用程序进程；   
（5）在左下角填上应用程序的真正入口点偏移（OEP）；  
（6）按"IAT AutoSearch"按钮，让其自动检测 IAT 位置， 出现"Found address which may be in the Original IAT.Try 'Get Import'"对话框，这表示输入的 OEP 发挥作用了。  
（7）按"Get Import"按钮，让其分析 IAT 结构得到基本信息；  
（8）如发现某个 DLL 显示"valid :NO" ，按"Show Invalids"按钮将分析所有的无效信息，在 Imported Function Found 栏中点击鼠标右键，选择"Trace Level1 (Disasm)"，再按"Show Invalids"按钮。如果成功，可以看到所有的 DLL 都为"valid:YES"字样；   
（9）再次刷新"Show Invalids"按钮查看结果，如仍有无效的地址，继续手动用右键的 Level 2 或 3 修复；  
（10）如还是出错，可以利用"Invalidate function(s)"、"Delete thunk(s)"、编辑 Import 表（双击函数）等功能手动修复。  
（11）开始修复已脱壳的程序。选择 Add new section (缺省是选上的) 来为 Dump 出来的文件加一个 Section(虽然文件比较大，但避免了许多不必要的麻烦) 。  
（12）按"Fix Dump"按钮，并选择刚在（2）步 Dump 出来的文件，在此不必要备份。如修复的文件名是"Dump.exe"，它将创建一个"Dump\_.exe"，此外 OEP 也被修正。  
（13）生成的文件可以跨平台运行。

 4dd466d0e1e0cc123afbe504da2462ca