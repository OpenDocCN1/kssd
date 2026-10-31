
# 转载：如何生成SoftICE可用的调试符号文件`*.NMS` (1千字)

除了可以直接从PE文件中提取出其Export Table中的函数符号名之外，SoftICE可以直接使用的调试符号文件只有`*.NMS`。

得到可执行程序的NMS文件的途径如下：

1. DBG/PDB－>NMS

   如果在编译时生成了`*.DBG`/`*.PDB`文件，直接用SoftICE Symbol Loader打开`*.DBG`文件，然后选择菜单Module->Translate即可生成`*.NMS`。在Translate选项中选择Include source则生成的`*.NMS`中包含有程序源码（当然前提是`*.DBG`/`*.PDB`中有源码），就可以在源码一级进行调试，否则NMS文件只包含符号名而没有源码。Windows系统本身的符号文件在其安装光盘上有。

   SoftICE自带的命令行工具（在其`util16`目录下）`dbg2map.exe`还可以将DBG转换成MAP文件，参考下面的2。

2. SYM－>NMS

   可以用SoftICE自带的命令行工具`nmsym.exe`将`*.SYM`转换成`*.NMS`。

3. MAP－>NMS

   如果没有可执行程序的源码，也没有`*.DBG`/`*.PDB`文件，可以先用IDA对程序进行反汇编，IDA可以识别出常用的库函数并生成`*.MAP`文件（链接器`link.exe`、`tlink.exe`也可以生成MAP文件，当然必须是自己写的程序）。再利用SoftICE自带的命令行工具`msym.exe`将MAP转换成SYM文件，然后参考上面的2得到NMS文件。

对于没有源码的COM组件，还可以利用Matt Pietrek写的`TLBDBG`（http://www.microsoft.com/msj/0399/code/TLBDBG.zip）根据其TypeLib得到MAP/DBG文件，进而再得到NMS文件。注意`TLBDBG`只能处理进程内COM服务器。

当然上述文件格式的转换还可以用其它工具来完成。

要加载`*.NMS`文件，只需在SoftICE Symbol Loader的菜单Edit->SoftICE Initialize Settings->Symbols中指定相应的NMS文件并重启即可。

作者；Netguy
