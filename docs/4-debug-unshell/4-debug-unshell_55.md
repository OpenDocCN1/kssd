

# 输入表重建工具 ReVirgin

> 原文：[`www.pediy.com/kssd/tutorial/chap8-5-4.htm`](https://www.pediy.com/kssd/tutorial/chap8-5-4.htm)

 ReVirgin（简称 RV）   
  
功能：      重建 Import Table；寻找 OEP   
下载地址：  http://www.woodmann.com/fravia/index.htm   
具体地址是：http://www.woodmann.com/fravia/exe/revirgin.zip   
  
一、安装。   
  
1、自动安装：   
  
直接双击\*.msi 就会自动激活 Windows Installer 进行安装，这是最省事的办法。如果你的系统太老，可能需要到微软主页上免费下载 Windows Installer 的包。   
  
2、手动安装：   
  
可以用 WinRAR 将\*.msi 解开到某个目录中。然后把 tracer.dll、thread.dll 拷贝到%SystemRoot%目录下；对于 NT/2K/XP 系统，把 rvtracer.sys 拷贝到%SystemRoot%\\system32\\drivers 目录中。   
  
二、重建 IT   
  
1、首先选择被加壳的程序所对应的进程。如果找不到则点击 refresh 按钮。   
  
2、再找被加壳了的程序的 OEP。找 OEP 的方法主要有：用 IceDump 的/tracex；用冲击波；手动用 debugger 跟；利用各种编译器生成的可执行程序的 startup code 的机器码的 pattern 来找；利用 RV 自带的 tracer 来找。   
  
3、找到 OEP 之后将其填入 RV 的相应位置（注意要填 VA 而不是 RVA），然后点击“Fetch IAT”，RV 会自动分析出 IAT 的起始 RVA 和 Length。如果你觉得 RV 找到的不对，你也可以手动找到这个 RVA 和 Length 并填入该位置。   
  
4、然后点击“IAT Resolver”按钮，RV 会自动分析，可能要等好几分钟。这时 CPU 占用率很高，最好不要切换到其它程序。   
  
5、分析完之后会看到一些 API 函数标记为“redirected/emulated”，此时点击“Resolve again”，大部分函数都可以 resolve 出来。   
  
6、然后在下拉列表中选择“Show unresolved”，因为此时只关心尚未分析出来的 API 函数。   
  
在函数列表框中选中一个或多个函数（按住 shift 可以选多个），然后打开右键菜单。对于每个未分析出来的 API 函数，你可以试试右键菜单中的“tracer”或“API Emulator”，如果 RV 能够分析出来，则相应的行会变成“traced”或“emulated”状态，并且 Address 这一列会指向 DLL 的地址范围。   
  
一旦变成“traced”或“emulated”状态，则你可以再次点击“resolve again”按钮，这些 API 函数将会被分析出来。   
  
注：   
a、未分析出来的函数很多时，最好不要用右键菜单中的“Trace all”，后果你试试就知道了；   
b、右键菜单中的 edit 是个开关，表示列表中的每列是否可以编辑，此时可以手动输入 API 函数的相关信息（但这里似乎有 bug，因为即使你手动把未分析出来的函数都手动填入了，RV 仍说没有全部分析完）。   
c、右键菜单中的 Tracer 功能可能会导致被跟踪的程序出现非法操作，所以最好是随时使用“Save resolved”按钮的功能把阶段性结果存为文本文件，一旦出现非法操作还可以用“Load resolved”功能把以前的结果加载进来继续干活。   
  
7、对所有的 API 函数重复 5、6 两步，大部分都可以分析出来。剩下的那些基本上要用 debugger 来手动分析了。   
  
8、所有的 API 函数都分析出来之后，就可以生成 IT 信息，并粘贴到脱壳后的文件中。首先你得确定把生成的 IT 放在程序的什么位置，一般是放在末尾（此时要添加一个 section），但实际上可以放在任何合理的位置。   
  
把存放 IT 的位置的 RVA 填入，并点击“generate”按钮。如果选中了“Auto fix sections + IT Paste”，则 RV 会问你脱壳后的 exe 文件名并自动把 IT 粘贴进去（但目前的版本没有帮你修改程序的 OEP，你得手动修改粘贴好的程序的 OEP），并生成一个 BIN 文件，这个 BIN 文件是由 IT、IAT、DLL 名和函数名组成的，供你手动粘贴用（如果你喜欢手动粘贴的话）。   
  
至此重建 IT 完毕。   
  
三、其它说明：   
  
1、RV 的 mangled scheme 选项是用来对付那种将多个 API 重定向到同一个函数的壳的，一般用不上。   
2、RV 的底部的“Tracer”按钮是用来跟踪程序找 OEP 的，功能也很强。 只需要指定 OEP 可能存在于什么范围（给出最小值、最大值），当被跟踪的程序的 EIP 落在此区间时 RV 就会停下来。这和利用 IceDump 的/tracex 来找 OEP 有些相似。   
3、API 函数列表中的 Refs 这一列是该函数的引用计数。   
4、最好是在 OEP 处将被加壳的进程 suspend，然后用 RV。因为某些被 ASProtect 加壳的程序在进入 OEP 之后会修改 IAT 的某些项。 

 4dd466d0e1e0cc123afbe504da2462ca