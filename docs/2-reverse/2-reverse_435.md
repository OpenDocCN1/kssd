

# IDA\_Pro\_plugin\_wizard 的一个 bug 及修正

> 原文：[`www.pediy.com/kssd/pediy10/94577.html`](https://www.pediy.com/kssd/pediy10/94577.html)

转自己的贴，原发于：  
http://www.team509.com/modules.php?name=News&file=article&sid=79  
  
以前写 IDA 的 plugin 比较烦，都要手工设置很多东西，比较烦。具体设置方式可以见 IDA PLUG-IN WRITING IN C/C++，3. Setting up a Build Environment  
以 VS 为例，设置过程如下：  
1 Go to File->New->Project… (Ctrl-Shift-N)  
2 Expand the Visual C++ folder, followed by the Win32 sub-folder, and then select the  
Win32 Project icon. Name the project whatever you like and click OK.  
3 The Win32 Application Wizard should then appear, click the Application Settings  
link on the left and make sure Windows Application is selected, and then tick the  
Empty Project checkbox. Click Finish.  
4 In the Solutions Explorer on the right hand side, right click on the Source Files  
folder and go to Add->New Item...  
5 Select C++ File (.cpp) under the Templates section and name the file appropriately.  
Click Add. Repeat this step for any other files you want to add to the project.  
6 Go to Project->projectname Properties...  
7 Change the following settings (some have been put there to reduce the size of the  
resulting plug-in, as VS seems to bloat the output file massively):  
Configuration drop down in the top left: Select Release  
Configuration Properties->General: Change Configuration Type to  
Dynamic Library (.dll)  
C/C++->General: Set Detect 64-bit Portability Issues checks to No  
C/C++->General: Set Debug Information Format to Disabled  
C/C++->General: Add the SDK include path to the Additional Include  
Directories field. e.g. C:\\IDA\\SDK\\Include  
C/C++->Preprocessor: Add \_\_NT\_\_;\_\_IDP\_\_ to Preprocessor Definitions  
C/C++->Code Generation: Turn off Buffer Security Check, set Basic  
Runtime Checks to Default and set Runtime Library to Multi-threaded  
C/C++->Advanced: Calling Convention is \_\_stdcall  
Linker->General: Change Output File from a .exe to a .plw in the IDA plugins  
directory  
Linker->General: Add the path to your libvc.wXX to Additional Library  
Directories. e.g. C:\\IDA\\SDK\\libvc.w32  
Linker->Input: Add ida.lib to Aditional Dependencies  
Linker->Debugging: No to Generate Debug Info  
Linker->Command Line: Add /EXPORT:PLUGIN  
Build Events->Post-Build Event: Set Command-line to your idag.exe to start  
IDA after each successful build (Optional)  
Click OK  
8 Go to Build->Configuration Manager… and change the drop-down in the  
Configuration column for your plug-in project from Debug to Release. Click OK  
9 Begin to write plugin  
够烦吧！  
  
现在好了，最近发现一款 IDA Pro plugin wizard 的工具，安装一下，就能在 VS 里很方便的编写 IDA 的 Plugin 了，而且你设置好 IDA 的安装目录之后，编译好的 plugin 还会自动复制到你的 IDA 安装目录的 plugins 目录里，哈哈，实在是方便啊:-)  
这个东东你可以去 http://jeru.ringzero.net/?page\_id=4 下载。  
  
这个东东有一个小小的 bug:  
  
如果你的 IDA 是按默认路径安装的，比如:C:\\Program Files\\IDA  
由于复制编译好的 plugin 文件时是用一句  
copy /y XXXXX.plw C:\\Program Files\\IDA\\plugins  
完成的，由于目标路径没有加上冒号，所以只要这个路径里有个空格，这条命令就死翘翘了:-(  
自然，编译好的 plw 文件也就去不了 IDA 的 plugins 目录了。  
  
解决方法有一个，那就是对这个东西略微修改一下，我一 VS2005 为例，做一个说明:  
\\IDA\_Plugin\_Wizard\_VS2005\\Put Contents into AppWiz\\IDA Pro Plugin\\Scripts\\1033  
这个目录下有一个 default.js，这个文件的第 246 行  
         idaBinDir = wizard.FindSymbol("BINPATH") + '\\\\plugins';  
显然这一句就是生成有问题的这个路径的，把这句改成  
         idaBinDir = '\\"' + wizard.FindSymbol("BINPATH") + '\\\\plugins\\"';  
就可以了。