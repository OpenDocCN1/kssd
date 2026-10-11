
# Armadillo 4.1 的使用说明

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-828.htm`](https://www.pediy.com/kssd/pediy08/pediy8-828.htm)

1. 首先准备好要加壳的目标程序或资源文件，如`exe`/`scr`/`dll`/`ocx`。
2. 运行`Armadillo`创建一个工程：`File->New Project...`。
3. 在`Project Settings`窗口的左边为操作项目，右边为各对应项目的具体内容：
   - 第一步：`Project ID and Version`项目：在右边此工程的`Project ID`，假设为`abc`。
   - 第二步：`Files to Protect`项目：`Primary File to Protect`为主要要进行保护的文件，假设添加`abc.exe`。如果还有`abc.exe`还有调用其它重要的扩展资源如`dll`文件，可以在`Secondary Files to Protect`中进行逐一的添加。
   - 第三步：`Splash Screen 1`项目：为启动软件时要显示的信息框，有三种方式，第一种为`No Splash Screen`即不显示任何信息，第二种为`Show Default Box`即显示默认的信息框 (会出现一个载入进度)，第三种为`Show This Bitmap`即可定义信息的背景图。在这里我们选择`No Splash Screen`选项。
   - 第四步：`Protection Options`项目：为保护的方式，有默认方式和双进程保护方式(前三种为双进程)，依个人喜爱自行选择，这里我们选择`Standard protection plus Debug-Blocker`。
   - 第五步：`Other Options`项目：为其它附加的保护方式，这里我们不选择。
   - 第六步：`Certificates`项目：单击`New`在`Certificate Settings`窗口中只需要设置`Certificate Name`证书的名称如`abc`并选择下面复选框中，这里我们选择默认方式`Set as Default Certificate`后就可以将所有打开的设置窗口关闭了（窗口右上角的关闭按钮）。

\--------------------------------------------加注册壳步骤-----------------------------------

   - 第六步: `Certificates`项目：单击`New`在`Certificate Settings`窗口中只需要设置`Certificate Name`证书的名称如`abc`并选择下面复选框中，在`Encryption Template`里增入你想要加入的加密代码，在选择方式`Use Digital River Edition Keys`，在把`Signature Level`里选`Level10`和`Allow Key String(Pro)`。
   - 第七步: `Hardware Locking(Pro)`里选`Standard hardware locking`后就可以将所有打开的设置窗口关闭了（窗口右上角的关闭按钮）。

\--------------------------------------------加注册壳步骤-----------------------------------

4. 开始加壳工作：`Protecton->Protect File...`就会出现一个加壳过程的窗口，成功后出现`Success`，点击`Close`返加主界面，此时如果你要修改加壳项目，这时你可以先通过`UnProtect File...`来恢复原来的程序再复新加壳。
5. 通过`File->Save Protect`可以将此工程进行保存，以便日后进行修改。
6. 补充一下：默认的压缩率为`Better/Slower Compression`，如果要改变压缩率可在`Compression Options`项目中进行更改，`Best/Slowest Compression`为最高压缩率，注意压缩率越高程序运行的速度就越慢。还有如果你的易语言程序采用独立编译的话，最后面有一个复选框`Compress Resources (Pro)`最好不要选！

`4dd466d0e1e0cc123afbe504da2462ca`
