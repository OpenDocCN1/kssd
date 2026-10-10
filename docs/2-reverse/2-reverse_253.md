

# 调试符号文件\*.NMS

> 原文：[`www.pediy.com/kssd/pediy03/forum831.htm`](https://www.pediy.com/kssd/pediy03/forum831.htm)

除了可以直接从 PE 文件中提取出其 Export Table 中的函数符号名之外，SoftICE 可以直接使用的调试符号文件只有\*.NMS。   
  
得到可执行程序的 NMS 文件的途径如下：   
  
1、DBG/PDB－>NMS   
如果在编译时生成了\*.DBG/\*.PDB 文件，直接用 SoftICE Symbol Loader 打开\*.DBG 文件，然后选择菜单 Module－>Translate 即可生成\*.NMS。在 Translate 选项中选择 Include source 则生成的\*.NMS 中包含有程序源码（当然前提是\*.DBG/\*.PDB 中有源码），就可以在源码一级进行调试，否则 NMS 文件只包含符号名而没有源码。Windows 系统本身的符号文件在其安装光盘上有。   
  
SoftICE 自带的命令行工具（在其 util16 目录下）dbg2map.exe 还可以将 DBG 转换成 MAP 文件，参考下面的 3。   
  
2、SYM－>NMS   
可以用 SoftICE 自带的命令行工具 nmsym.exe 将\*.SYM 转换成\*.NMS。   
  
3、MAP－>NMS   
如果没有可执行程序的源码，也没有\*.DBG/\*.PDB 文件，可以先用 IDA 对程序进行反汇编，IDA 可以识别出常用的库函数并生成\*.MAP 文件（链接器 link.exe、tlink.exe 也可以生成 MAP 文件，当然必须是自己写的程序）。再利用 SoftICE 自带的命令行工具 msym.exe 将 MAP 转换成 SYM 文件，然后参考上面的 2 得到 NMS 文件。   
对于没有源码的 COM 组件，还可以利用 Matt Pietrek 写的 TLBDBG（http://www.microsoft.com/msj/0399/code/TLBDBG.zip）根据其 TypeLib 得到 MAP/DBG 文件，进而再得到 NMS 文件。注意 TLBDBG 只能处理进程内 COM 服务器。   
  
当然上述文件格式的转换还可以用其它工具来完成。   
  
要加载\*.NMS 文件，只需在 SoftICE Symbol Loader 的菜单 Edit－>SoftICE Initialize Settings－>Symbols 中指定相应的 NMS 文件并重启即可。   
  
作者；Netguy  4dd466d0e1e0cc123afbe504da2462ca