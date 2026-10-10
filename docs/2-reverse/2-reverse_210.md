

# winxp+vpc2007+win2003sp1+windbg

> 原文：[`www.pediy.com/kssd/pediy10/80082.html`](https://www.pediy.com/kssd/pediy10/80082.html)

一下是学习 wrk 初期整理的关于运行环境的配置  
一.安装 dvd-windows  
  1. 解压并用虚拟镜像打开。  
  2. 运行 WRK-CRK 中的 wrkinstall.bat(推荐的安装方式，一些参数配置较方便)。  
     在 C:\\WRK-v1.2 中会默认安装了 wrk 的源文件。  
  3. 安装 WRK-CRK\\Virtual PC 2007 中的 virtual pc 2007(最好默认安装).  
     按向导创建一个虚拟机。  
  4. 安装 WRK-CRK\\WinDbg 中的 windbg(最好默认安装).  
二.配置 virtual pc 2007 并运行 win2003sp1  
  1. 开始-所有程序-macrosoft virtual pc.  
  2. setting-hard disk1-virtual hard disk file-browse:C:\\WRK-v1.2\\Win2k3SP1 Image\\Win2k3 SP1 WRK Hard Disk.vhd  
  3. com1-named pipe:\\\\.\\pipe\\debug  
  4. ok.退出 setting。  
  5. 回到 virtual pc console,start,在 command prompt 中选 windows 2003 sp1 standard.在虚拟机上启动了 win2003 sp1.  
三. recompile wrk  
  1. 通过主机与虚拟机之间的共享磁盘 c 盘，将主机上 C:\\WRK-v1.2 靠到虚拟机上 c 盘中。  
  2. 在虚拟机上，可以修改 C:\\WRK-v1.2\\base\\ntos 中相应内核程序。  
  3. start-all program- Accessories-command prompt.path \\wrk-v1.2\\tools\\x86;%path%(设置编译环境文件路径) cd c:\\WRK-v1.2\\base\\ntos 切换路径，然后 nmake x86=，从编译开始，大约需要几分钟的时间。  
  
  4. 编译完后在 C:\\WRK-v1.2\\base\\ntos\\BUILD\\EXE 下会生成一个 wrkx86.exe 和一个 wrkx86.pdb，将 wrkx86.exe 拷到虚拟机的 C:\\WINDOWS\\system32\\下。将 wrkx86.pdb 靠的主机的符号文件的目录下。可能会有重名文件，可以重新命名，但在修改 boot.ini 时，要保持一致。  
  5. 将主机 C:\\WRK-v1.2\\WS03SP1HALS 下的 hal 文件复制到虚拟机 C:\\WINDOWS\\system32 下。  
  6. 通过下述设置 boot.ini 可以重启 win2003 sp1.运行新编译的内核。  
四. 配置 windbg，并调试内核。  
  1. 在主机上打开 command prompt,C:\\Program Files\\Debugging Tools for Windows\\,切换路径。  
  2. 执行 windbg.exe k com:port=\\\\.\\pipe\\debug,pipe,resets=0,reconnect  -y C:\\Symbols\\wrk（-y 为设置符号文件路径，三中 4 提到的符号文件），会打开 windbg 窗口，显示正在连接。  
(1，2 可以用 windbg 图形界面来完成，打开 windbg-file-symbol source path,设置符号表位置；然后 file-kernel debug-com，设置 com 为\\\\.\\pipe\\debug，选中 pipe 和 reconnect，resets=0；）  
  3. 打开虚拟机，运行 win2003 sp1,选择 debugger enable 的那个。然后 windbg 就会显示连接成功，可进行调试内核。  
  
五. 在虚拟机上设置 boot.ini.  
  1. my computer 右击-properties-system properties-advanced-startup and recovery-setting-startup and recovery-edit,在 notepad 中打开 boot.ini.  
  2. edit boot.int:在 operating system 一栏最后添加  
multi(0)disk(0)rdisk(0)partition(1)\\WINDOWS="Windows Server 2003, Enterprise" /fastdetect/DEBUG/DEBUGPORT=COM2/BAUDRATE=115200/kernel=wrkx86.exe/hal=halmacpi.dll  
  3. 保存-确定。  
  
注：  
 出现问题：运行.bat 文件时出现   没有找到 MSVCP71.dll 因此这个应用程序未能启动。重新安装应用程序可能会修复此问题  
           只需下载 MSVCP71.dll 放入 windows(winnt)\\system32 下  
           修改内核代码要改文件内容，不可 copy。否则编译系统不认为它是修改过的，不予编译。