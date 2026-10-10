

# trw2000 vb 符号调试初步

> 原文：[`www.pediy.com/kssd/pediy05/pediy50204.htm`](https://www.pediy.com/kssd/pediy05/pediy50204.htm)

trw2000 vb 符号调试初步   
  
    我们都知道 softice 支持符号调试。而很少了解 TRW2000 也具有自动载入 sym 符号文件供调试的作用。   
    为了对 Visual Basic 6 进行符号调试，首先需要得到 msvbvm60.dll 及相对应的 msvbvm60.dbg 调试文件，这些可以从 microsoft 网站的 VB6 SP5 升级包中得到。   
    在你安装 softice 的目录\\Util16 下，有 2 个程序：Dbg2map 及 msym.exe   
dos 窗口下命令   
c:\\..\\softice\\util16> dbg2map.exe msvbvm60.dbg  生成 msvbvm60.map 文件   
c:\\..\\softice\\util16> msym.exe    msvbvm60.map  生成 msvbvm60.sym 文件   
  
    将 msvbvm60.sym 文件放在 trw2000.exe 所在目录下，运行 trw 时会自动载入 sym 文件。   
由此我们可以得到 msvbvm50.sym、oleaut32.sym 文件并一并载入。当多个 sym 被载入后可以在 trw2000 中下命令   
sym   
来选择当前使用的 sym 文件。   
  
    所以，我们将可以直接用 exdec 反编译得到的 vb pcode 助记符设断拦截。也可以在用 trw 进行跟踪时获得更多的程序信息。   
  
    注意：   
    1. 个人使用后体会，trw2000 1.23 版本中，利用 vb pcode 助记符设断经常会出错，表现在 trw 不能将助记符正确转换为中断地址。   
  
    2. 如此也解决一个问题：exdec 反编译后生成的 pcode 助记符名称是谁定义的？   
      答案是：微软公司  4dd466d0e1e0cc123afbe504da2462ca