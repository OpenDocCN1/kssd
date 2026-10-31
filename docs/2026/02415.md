
# 斗地主v2.4密码的最简单破解――OllyDbg破解法

作者：大唐飞侠  
时间：2001年8月1日  
使用工具：`OllyDbg`  
下载地址：`http://home.t-online.de/home/Ollydbg/odbg104.zip`  
软件名称：斗地主 v2.4  
下载地址：`http://soft.nyinfo.ha.cn/down1.php?id=5910&url=http://61.177.64.99/down/ddz1.zip`

## 破解步骤

1. 运行`OllyDbg`，按`F3`加载`DDZ.EXE`，选择"Deebug" 并且点击"Run"或按`F9`运行`DDZ.EXE`，DDZ运行了...... 怎么了斗地主界面不肯出来，不要骂我，是`OllyDbg`捣的鬼，点击`OllyDbg`界面，按住`Shift`同时按几下`F9`就可以回到斗地主界面。

2. 设置断点，回到`OllyDbg`界面，通常`OllyDbg`界面有四个窗口，右击左上角那个窗口，选择“`Search for ->Name (label)`”出现软件所调用函数的窗口，因为DDZ是VB程序，向下拉动滚动条找到VB函数`__vbaStrComp`并选择它，按下`F2`左边变成红色，断点设置好了。

3. 象往常一去注册吧，填  
   注册名：`datangfx`  
   注册码：`7086708670`  
   点击确定，程序马上被`OllyDbg`拦住了，停在地址`6602416C`这里，右下角窗口显示我的大名 UNICODE“datangfx”，好了这下有戏看了，按一下`F9`，显示我输入的注册码“7086708670”，再次按`F9`，真的注册码出来了“0174109044”，哈哈。。。  

   整理注册信息  
   注册名：`datangfx`  
   注册码：`0174109044`  
   用`OllyDbg`破解，过程就是那么简单，不信你试一下。

转载时希望保持完整，谢谢！
