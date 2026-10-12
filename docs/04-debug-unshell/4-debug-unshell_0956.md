
# RLPack v1.21 Full OEP Finder + Fix IAT Script v0.1

> 原文：[`www.pediy.com/kssd/pediy10/87563.html`](https://www.pediy.com/kssd/pediy10/87563.html)

这个脚本是我自己写了来脱 `RLPack v1.21 Full` 主程序壳的。修复虚拟机我用的是 `SND` 论坛上 `LCF-AT` 发的 `RLPack 1.21 VM Code Translater 1.0` 脚本，在此表示感谢！`LCF-AT` 的原帖链接在这里：

[`forum.tuts4you.com/index.php?showtopic=19712`](http://forum.tuts4you.com/index.php?showtopic=19712)

`LCF-AT` 发的 这个 VM 修复脚本很不错，我已经放在附件中了。下面对我写的这个脚本作一些说明：

### 一、脚本针对的是使用了 RLPack v1.21 Full 一些高级保护选项的程序，目前的功能主要有：

1. 移除壳所有的 `Anti`，没修改过的 `OD` 应该也可以调试；
2. 自动在日志中列出原始程序的所有区段；
3. 自动查找 `OEP`，若有 `Stolen Code` 的话，自动给出 `Stolen Code` 表达式的开始地址，可以在数据窗口定位到这个地址，显示方式选“长型->地址”，参考各个参数来修复 `Stolen Code`。这里是八个字节的参数代表一个指令，如这样的：

   ```
   00C60000  00000003
   00C60004  00000007
   ```

   表示指令 `push ebp`。这些参数我没研究完，也没空看了。哪位有时间可以研究一下。
4. 自动修复输入表，如果 `Delphi` 类的程序使用了 `RLPack` 的 `Import Elimination protection` 选项保护的话，会自动修复 `IAT` 的跳转表。

### 二、已知问题：

1. 如果 `delphi` 类的程序使用了 `RLPack` 的 `Thread Local Storage Protection` 选项的话，不会自动恢复 `TLS` 表。以后有精力再完成这一部分。
2. 可能支持 `DLL`，没测试。
3. 断断续续写的，脚本看起来可能有点乱。

### 三、使用方法：

`ODBGScript` 插件需要 `1.65` 以上。载入加壳的程序后先跑我的脚本，跑完如果有 `Stolen Code` 的话，根据提示修复。有 `VM` 的话，根据日志窗口中看到的 `OEP` 地址，`CTR+G` 转到 `OEP`，右键选择“此处为新 `EIP`”，然后跑 `LCF-AT` 的 `VM` 修复脚本。这个可能要有一段时间。等脚本完成后 `dump`，修复输入表。若有 `TLS` 的话，则要自己修复。

### 四、附件压缩包中的文件说明：

- `RLPack.v1.21 Full OEP Finder + Fix IAT.txt`：我写的脚本
- `RLPack 1.21 VM Code Translater 1.0.txt`：`LCF-AT` 的 `VM` 修复脚本
- `OEP BYTES.txt`：`LCF-AT` 写的 `RLPack v1.21 Full` 主程序入口点被抽的代码
- `RLPack.exe`：我脱过壳的 `RLPack v1.21 Full` 主程序

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `RLPack.1.21.Full.Unpacked.rar`  
|
