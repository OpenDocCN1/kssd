
* 创建`kerenel32.def`文件句柄为`0x00000080`。(如图35,36)

  ![图片描述](upload/attach/201804/510904_K63ZG2YKTFTJHJJ.png)

  ![图片描述](upload/attach/201804/510904_3ZKGT7QSB2Q9GEF.png)

* 创建`kerenel32.asm`文件句柄为`0x00000084`。(如图37,38)

  ![图片描述](upload/attach/201804/510904_49JKVCPUT4MUBCS.png)

  ![图片描述](upload/attach/201804/510904_WD9743D6KQFB2VN.png)

* 多次往`Kernel32.asm`写数据代码。(如图39)

  ![图片描述](upload/attach/201804/510904_T6XTEQEJXGCWXPW.png)

* 往`kerenl32.def`写代码。(如图40)

  ![图片描述](upload/attach/201804/510904_AJS5Q2PC4GSBMFK.png)

* 目录下创建了`kernel32.asm`和`kernel32.def`文件,并且写入很多数据。(图41)

  ![图片描述](upload/attach/201804/510904_83NKMWFTZQ6ERQK.png)

* 根据以上行为分析:读`inc`, 写`asm`,`def`文件。按照一般思路,有了`asm`,`def`文件,想要生成`lib`，一般都会使用微软提供的`ml`/`cl`程序(至少我是这样的)。如果在程序调用这些程序,那么应该会有`CreateProcess`来创建进程。因此,我们在`CreateProcessA`,`CreateProcessW`来创建进程。(如图42,43,44)

  ![图片描述](upload/attach/201804/510904_U37ZJDX5U9QZVNV.png)

  ![图片描述](upload/attach/201804/510904_YMAPZR2SUC7P8G9.png)

  ![图片描述](upload/attach/201804/510904_XQ2VTMTG6XJAFQW.png)

* 为了查看详细的错误代码,我们写个`DEMO`程序,模拟`inc2l`的`CreateProcess`，输出错误原因。

```c
#include <stdio.h>
#include <stdlib.h>
#include <windows.h>
void ShowErrorMsg();
int main(int argc,char \*argv[])
{
char szCommandLine[] = "\\masm32\\bin\\ml /c /coff kernel32.asm";
STARTUPINFO si = { sizeof(si) };
PROCESS\_INFORMATION pi;
si.dwFlags = STARTF\_USESHOWWINDOW; //指定wShowWindow成员有效
si.wShowWindow = TRUE; //此成员设为TRUE的话则显示新建进程的主窗口
BOOL bRet = CreateProcess(
NULL, //不在此指定可执行文件的文件名
szCommandLine, //命令行参数
NULL, //默认进程安全性
NULL, //默认进程安全性
FALSE, //指定当前进程内句柄不可以被子进程继承
NORMAL\_PRIORITY\_CLASS,
NULL, //使用本进程的环境变量
NULL, //使用本进程的驱动器和目录
&si,
&pi
);
if (bRet)
{
CloseHandle(pi.hThread);
CloseHandle(pi.hProcess);
}
else
{
ShowErrorMsg();
}
system("pause");
return 0;
}
void ShowErrorMsg()
{
LPVOID lpMsgBuf;
FormatMessage(
FORMAT\_MESSAGE\_ALLOCATE\_BUFFER |
FORMAT\_MESSAGE\_FROM\_SYSTEM |
FORMAT\_MESSAGE\_IGNORE\_INSERTS,
NULL,
GetLastError(),
MAKELANGID(LANG\_NEUTRAL, SUBLANG\_DEFAULT), // Default language
(LPTSTR)&lpMsgBuf,
0,
NULL
);
MessageBox(NULL, (LPCTSTR)lpMsgBuf, TEXT("Error"), MB\_OK | MB\_ICONINFORMATION);
LocalFree(lpMsgBuf);
}
```

* 将生成的程序拷贝到`inc2l_dump.exe`同目录下。发现错误原因系统找不到指定文件。(如图45)

  ![图片描述](upload/attach/201804/510904_X2JNMMD8YCHV3T3.png)

* 是不是`CreateProcess`中路径参数`\masm32\bin\ml`的错误呢,将`DEMO`程序中的路径换成相对路径再试试。再将`DEMO`程序和`kernel32.asm`拷贝到`\masm32\tools\inc2l`目录下运行。运行正常还生成了`obj`文件。(如图46,47)

  ![图片描述](upload/attach/201804/510904_QCUXEGX5YC74TUN.png)

  ![图片描述](upload/attach/201804/510904_PY6MEPBHHUEEN84.png)

* 经过测试,相对路径错误,绝对路径正确。那么说明是环境变量的问题。猜想`inc2l`中是不是需要哪个环境变量指向`\masm32\bin`里呢。抱着疑问,我们继续回到`OD`调试。通过桟回溯找到取环境变量的位置。(如图48)

  ![图片描述](upload/attach/201804/510904_HN7H4ZWA24S6YSN.png)

* 有两个跳转,拼接的命令行参数不同。(如图49,50)

  ![图片描述](upload/attach/201804/510904_MJKQKD4WTQJH58X.png)

  ![图片描述](upload/attach/201804/510904_7WHVJSWAQ29RNEH.png)

* 大致跟进去查看后,猜测大致流程如下图(天生艺术细胞不强,凑合着看吧):(如图51)

  ![图片描述](upload/attach/201804/510904_7VKTMCC8KQ62H5N.png)

* 既然是因为取环境变量`mdir`,而我们又没有这个环境变量。那是不是加上了环境变量就正常了呢。加个环境变量测试下。(如图52,53)

  ![图片描述](upload/attach/201804/510904_GW7E87WVYJD5N47.png)

  ![图片描述](upload/attach/201804/510904_SK6H3SJ2CQEKSWS.png)

## 6.2 分析总结

* 执行流程

  1. 打开读取参数文件。

  2. 创建`参数.def`文件和`参数.asm`文件。

  3. 根据参数写`参数.def`文件和`参数.asm`文件

  4. 取环境变量`mdir`，如果取到则和`\bin\ml`与`bin\link`组合编译链接。如果没有取到环境，则用默认的`masm32\bin\ml`与`masm32\bin\link`组合编译链接。

* 因为默认的没有盘符,所以`inc2l.exe`必须与`masm32`处于同一盘下才能正常使用。之前我们将`inc2l.exe`复制到D盘,而`masm32`安装在C盘,因此没有正确打包成`LIB`文件。为了验证我们的猜想,我们将`mdir`环境变量删除,并在`inc2l_dump.exe`所在的D盘根目录新建文件夹`masm32`,在`d:\masm32`中新建文件夹`bin`,并且将`ml.exe`、`link.exe`拷贝其中。再尝试编译。(如图54)

  ![图片描述](upload/attach/201804/510904_QX57BKQTYM88RVR.png)

* 我们已经从上实验中得知,之所以没有打包成功,是因为编译链接时候找不到`ml.exe`与`link.exe`。而我们在安装`VC6`和`masm32`后,已经把`ml.exe`与`link.exe`加入环境变量中(如图55)

  ![图片描述](upload/attach/201804/510904_CDAB5HX2N2U7P2R.png)

* 那么我们在`inc2l_dump.exe`没有取到环境变量`mdir`,则直接使用`ml.exe`和`link.exe`编译链接呢?(即将`masm32\bin\ml`与`masm32\bin\link`修改为`ml`与`link`)。`OD`中看到默认的`masm32\bin\ml`与`masm32\bin\link`全局字符串,那么我们打开`WinHex`定位到此处。(如图56)

  ![图片描述](upload/attach/201804/510904_P2Z66KRWSZN2QNJ.png)

* 注意,此处用的是`Pascal`,前4字节所以放长度,后面放字符串,0结尾。所以修改时注意。(如图57)

  ![图片描述](upload/attach/201804/510904_6KQ9XHWNRPJ3NHU.png)

* 此时再使用`inc2l`即可正常将`inc`文件打包成`lib`文件。(如图58)

  ![图片描述](upload/attach/201804/510904_EGH6UPNQ75JVECS.png)

* 总结修复方案

  1. 设置环境变量`mdir`

  2. 保证`inc2l.exe`与`masm32`处在同一盘下。

  3. 修改`inc2l.exe`二进制,直接使用`ml.exe`、`link.exe`编译链接。

* 其实这个环境变量的问题也不好认定是否是`BUG`,说它是`Bug`,它在帮助里给出了提示,指出了要依赖环境变量`mdir`。但说它不是`BUG`,这提示藏得有点深,一般人机子上都不会默认有`mdir`这个环境变量吧。而且如果因为路径原因调用`ml.exe`和`link.exe`失败,至少得给个提示吧,因为`XXX`原因生成失败...这直接就是空白无任何提示。用户体验相当不好啊.

  ![图片描述](upload/attach/201804/510904_R7KNV2JQVUW786E.png)

# 七.Hello World

* 将打包好的`kernel32.lib`替换`C:\masm32\lib\kernel32.lib`。(如图59)

  ![图片描述](upload/attach/201804/510904_HDWJ5KYSJ25CSZU.png)

* 尝试编译链接已经成功,但运行依旧出问题。(如图60)

  ![图片描述](upload/attach/201804/510904_U9ZGM2GRMJ7QKW4.png)

* 这是因为我们给`RadAsm`提供的`.obj`是`VC`环境下编译的,因此对应的入口函数也应该改为`mainCRTStartup`。(如图61)

  ![图片描述](upload/attach/201804/510904_EJF8PZHDAWVAUDX.png)

* 运行编译出的`EXE`,久违的`Hello World`。终于出来了。(如图62)

  ![图片描述](upload/attach/201804/510904_X3Q9PJ4QACU2PWW.png)

# 一.前言

**近日在科锐学完了汇编课程,练手之余发现了由`VC6`编译出`obj`文件与汇编联合编译时候引发的`BUG`,在解决的过程中再次发现另外两个`BUG`.此文由笔记整理而成,虽力求完美,却因小弟才疏学浅难免有不足之处,希望各位朋友点评,让小弟进步.这篇笔记由初见`BUG`,`ESP`定律脱壳,重建导入表,调试分析`inc2l`,修复`inc2l`这几部分构成.笔记结构如下:**

- 初见BUG

  - 使用`VC6`编译出的`obj`文件与汇编联合编译。因`VC6`使用的`kernel32.lib`和`masm32`使用的`kernel32.lib`对`GetEnvironmentStrings`这个函数的声明不一致引出第一个`BUG`。

  - 修复`masm32`中`kernel32.INC`后,在使用工具`inc2l`将`kernel32.inc`打包为为`kernel32.lib`时,引出第二个`BUG`。(`inc2l`打包不成功,且没有任何提示)

- ESP定律脱壳

  - 在分析`inc2l`之前从查壳软件中得知`inc2l`加了压缩壳,使用`ESP`定律脱壳。

- 重建导入表

  - 脱壳后,利用`OD`的`Dump`程序后,发现导入表已经被毁。使用工具`impREC`重建导入表引出了第三个`BUG`。(`impREC`会添加新节(`mackt`)存放节表,有的样本新增节表的位置刚好超过了`PE`中的`SizeOfHeaders`所限定的大小。恰恰`ImpREC`并没有更新`PE`中的`SizeOfHeaders`的大小。)(最新版`impREC1.7`依旧没有修复这个`BUG`)

- 分析Inc2l

  - 经过分析,发现`inc2l`会取环境变量`mdir`,如果取到则使用`\bin\ml`与`bin\link`组合编译链接。如果没有取到环境,则使用缺省的`masm32\bin\ml`与`masm32\bin\link`组合编译链接。但缺省的组合方式没有写明盘符,所以`inc2l`必须要和`masm32`在同一盘符下才能正常使用.

- 修复Inc2l

  - 从上文得知了`inc2l`出错的由来,可知修复方案为:

    1. 新建环境变量`mdir`.

    2. 保证`inc2l`与`masm32`处在同一盘符下.

    3. 修改`inc2l`的二进制,将缺省的组合方式修改为`ml.exe`和`link.exe`编译链接.

# 二.环境

1. `Microsoft Visual 6.0`

2. `RadASM`

3. `Win7 64bit`

# 三.初见BUG

## 3.1 新建工程

- 使用`VC`新建控制台工程`TestHello`,编写函数`ShowHello()`。

- 使用`RadAsm`新建汇编控制台工程`TestBug`后,将`C`语言`TestHello`工程编译的`Test.obj`文件复制到`TestBug`目录下,并拖到工程中.如图1.

  ![图片描述](https://bbs.kanxue.com/upload/attach/201804/510904_SRZZE9396CKAAXU.png)

- `Link`的时候提示两个错误,如图2:

  ![图片描述](https://bbs.kanxue.com/upload/attach/201804/510904_YW9KBJY8DTE3UPG.png)

## 3.2 Link错误一:缺少`_Main`符号

- 之所以提示缺少`_Main`符号这个错误,是因为在我们的`Test.obj`里使用了`Printr`,而`Printf`的实现在`LIBCD.LIB`里。所以我们`C`语言工程在生成`Test.obj`的时候将`LIBCD.LIB`一起`Link`。而在`LIBCD.LIB`里由一个`ctr0.obj`,其中包含了`VC6`的入口代码。我们可以用`IDA`或者桟回溯来验证猜想.

### 3.2.1 IDA验证缺少`_Main`符号原因

- 将`LIBCD.LIB`(`LIBCD.LIB`在`VC6`安装目录 > `VC98` > `LIB`中)复制到桌面。(因权限问题本人在安装目录下打开失败,所以将其复制到桌面打开.)如图3.

  ![图片描述](https://bbs.kanxue.com/upload/attach/201804/510904_HH3QKFC94EYTEWY.png)

- 双击`crt0.obj`进去,会看到入口代码,如图4.

  ![图片描述](https://bbs.kanxue.com/upload/attach/201804/510904_N4RA3AXE4CUG9S6.png)

- 向下翻,会执行一些初始化的工作,如图5.

  ![图片描述](https://bbs.kanxue.com/upload/attach/201804/510904_F36SMJB54CFDTVX.png)

### 3.2.2 栈回溯验证缺少`_Main`符号原因

- 打开`VC`工程,通过栈回溯窗口进入`WinMainCRTStartup`函数中。

- 在`WinMainCRTStartup`函数中会进行一些初始化的工作,如根据字符集选择`A`版或者`W`版的获取环境变量函数。最后根据条件编译调用`wmain`或者是`main`.(如图6).

  ![图片描述](https://bbs.kanxue.com/upload/attach/201804/510904_8EW78HVBVWBYEWM.png)

### 3.2.3 解决错误一:缺少`_Main`符号

- 由上可知是因为`Link`了`LIBCD.LIB`,而`LIBCD.LIB`中又由入口函数被`LINK`。但拷贝到`RadAsm`工程中的`Test.obj`并没有定义入口函数,缺少一个`main`的实现。而`Link`的流程又需要调用`main`符号。基于此,只要能检测到`main`符号,就能解决掉这个错误。有两个解决方案:

  - 解决方案一:定义`main`的空函数:(如图7)

    ![图片描述](https://bbs.kanxue.com/upload/attach/201804/510904_EMQXYFJH44ZEF4D.png)

  - 解决方案二:将工程中的程序入口点改为`main`:(如图8)

    ![图片描述](https://bbs.kanxue.com/upload/attach/201804/510904_6BTHTE5PRWC36FR.png)

## 3.3 Link错误二:`GetEnvironmentStrings`

- `Link`时的第二个错误提示没找到`GetEnvironmentStrings`函数实现,而`GetEnvironmentStrings`的实现在在`kernel32.lib`这个库中。

- 下面分别去看看`VC6`与`RadASM`使用`GetEnvironmentStrings`这个函数的声明.

  ```
  VC6使用的kernel32.lib库路径     : VC6安装目录/VC98/Lib/

  RadASM使用的kernel32.lib库路径  : masm32/Lib

  ```

### 3.3.1 ReadAsm -> `GetEnvironmentStrings`函数声明
