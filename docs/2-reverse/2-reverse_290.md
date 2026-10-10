

# SoftSnoop2009 应用一：如何跟踪加壳程序

> 原文：[`www.pediy.com/kssd/pediy10/101544.html`](https://www.pediy.com/kssd/pediy10/101544.html)

最新版本 SoftSnoop2009V0.3 已提供下载。  
  
简单的举例说明下：SoftSnoop2009 如何跟踪加壳程序  
  
被跟踪的程序：[cyclone.rar](http://bbs.pediy.com/attachment.php?attachmentid=34513&d=1258608250)（也是在看雪中下的）  
  
使用 SoftSnoop 跟踪程序有四种方式：  
1、“文件”--“打开”  
2、“附着到进程”   
3、拖放程序到 SoftSnoop 窗口  
4、“选项”--“常规”-- “注册 Shell 扩展” 选中，exe 类型右键中会有“debug with SoftSnoop”  
  
我们先使用第四种方式，跟踪程序直接报错：  
![](img/5d7c2881c95580f203f5511159a8dbd0.jpg)  
  
说明直接跟踪失败。使用 OD 运行可以过（海风月影的 StrongOD 真的强大，膜拜一下）。  
  
下面请使用 OD 来附助 SoftSnoop 跟踪此程序：  
第一步：使用 OD 打开 cyclone.exe  
第二步：打开 SoftSnoop，使用第二种跟踪方式“附着到进程”（晕，“附加到进程”好听点吧），把 cyclone.exe 附加到 SoftSnoop  
第三步：在 OD 里运行程序，可以运行，没有报错。可惜什么都没有跟踪到。  
  
没有跟踪到，有两种可能：  
第一种：程序后面的运行也不在主程序范围内  
第二种：SoftSnoop 有问题  
  
查看列举的模块，发现了“ZShell32.dll”，这是什么动态连接库。用百度查找，只找到“Zprotect，与你同行”，竟然还是看雪里的。不用看了，这肯定是壳生成的东西。  
  
在“设置”---“DLL 调用”---“报告这些模块内的 API”选中，加入 ZShell32.dll  
![](img/c3df81ff588f4f6cd739fe2c743e6fe2.jpg)  
  
重新操作前三步：用 OD 打开，附加，运行。跟踪列表出来了：（列表比较长就不全列了）  
  
返回地址: 010643A7  函数名称: OL\_n10024(ZShell32.dll)   
OL\_n10024 返回值: 0x00000001  
       
返回地址: 010643BE  函数名称: MessageBoxA(USER32.dll)   
MessageBoxA: 信息对话框  
          hWnd=0x001D02D4  
          lpText="无效的序列号！"  
          lpCaption="错误"  
          uType=0x00000010 (MB\_OK|MB\_ICONERROR|MB\_APPLMODAL)  
MessageBoxA 返回值: 0x00000001 (IDOK)  
  
总结：  
如果一个进程 SoftSnoop 不能直接跟踪，请使用 OD 附助操作。