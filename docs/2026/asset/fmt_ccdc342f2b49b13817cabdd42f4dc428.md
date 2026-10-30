
# [原创]使用winafl对迅雷的torrent解析逻辑进行fuzz

## 一、漏洞挖掘目标

打开torrent文件时，迅雷会对torrent文件进行解析，得到需要下载的内容，所以想要对迅雷解析torrent文件的逻辑进行尝试二进制漏洞挖掘。

## 二、目标模块与接口

经过使用`procmon`抓包以及逆向分析，发现迅雷对torrent文件的解析流程在动态链接库`AssistantTools.dll`的导出函数`XL_ParseTorrentFileW`中。

所以目标模块就是`AssistantTools.dll`，接口就是`XL_ParseTorrentFileW`。

## 三、部署winafl环境

1. `winafl`目录在`D:\\_\_workspace\_\_\fuzz\winafl`
2. `dynamorio`目录在`D:\\_\_workspace\_\_\fuzz\dynamorio`

注：编译`winafl`和`dynamorio`请参考[这篇文章](https://bbs.pediy.com/thread-261323.htm)

## 四、编写fuzz主程序并编译

### 1.源文件`fuzz_program.cpp`

```
#include <stdio.h>
#include <Windows.h>
#define ASSISTANTTOOLS\_DLL "AssistantTools.dll"
extern "C" \_\_declspec(dllexport) VOID fuzz\_method(CHAR \* filePath);
// AssistantTools.dll
static HMODULE hAssistantTools = NULL;
// AssistantTools!XL\_ParseTorrentFileW
signed int \_\_cdecl XL\_ParseTorrentFileW(CHAR\* aFileName, PVOID\* a1);
using Fun\_XL\_ParseTorrentFileW = decltype(&XL\_ParseTorrentFileW);
static Fun\_XL\_ParseTorrentFileW fun\_XL\_ParseTorrentFileW = NULL;
// AssistantTools!XL\_ReleaseTorrentFileInfoW
void \_\_cdecl XL\_ReleaseTorrentFileInfoW(PVOID a1);
using Fun\_XL\_ReleaseTorrentFileInfoW = decltype(&XL\_ReleaseTorrentFileInfoW);
static Fun\_XL\_ReleaseTorrentFileInfoW fun\_XL\_ReleaseTorrentFileInfoW = NULL;
VOID fuzz\_method(CHAR\* filePath)
{
PVOID a1 = NULL;
fun\_XL\_ParseTorrentFileW(filePath, &a1);
if (a1)
{
fun\_XL\_ReleaseTorrentFileInfoW(a1);
}
return;
}
int main(int argc, char\* argv[])
{
CHAR assistantTools\_path[MAX\_PATH] = {0};
CHAR\* pTemp = NULL;
if (2 != argc)
{
printf("Parameter Error\n");
goto End;
}
// 获取AssistantTools.dll的全路径
if(0 == GetModuleFileNameA(NULL, assistantTools\_path, sizeof(assistantTools\_path)))
{
printf("GetModuleFileNameA Error\n");
goto End;
}
pTemp = strrchr(assistantTools\_path, '\\');
if(NULL == pTemp)
{
printf("strrchr Error\n");
goto End;
}
++pTemp;
\*pTemp = '\0';
strcat(pTemp, ASSISTANTTOOLS\_DLL);
// 加载AssistantTools.dll并获取函数地址
hAssistantTools = LoadLibraryA(assistantTools\_path);
if (NULL == hAssistantTools)
{
printf("LoadLibraryA fail: 0x%x\n", GetLastError());
goto End;
}
fun\_XL\_ParseTorrentFileW = (Fun\_XL\_ParseTorrentFileW)GetProcAddress(hAssistantTools, "XL\_ParseTorrentFileW");
if (NULL == fun\_XL\_ParseTorrentFileW)
{
printf("GetProcAddress fail\n");
goto End;
}
fun\_XL\_ReleaseTorrentFileInfoW = (Fun\_XL\_ReleaseTorrentFileInfoW)GetProcAddress(hAssistantTools, "XL\_ReleaseTorrentFileInfoW");
if (NULL == fun\_XL\_ParseTorrentFileW)
{
printf("GetProcAddress fail\n");
goto End;
}
fuzz\_method(argv[1]);
End:
if(hAssistantTools)
{
FreeLibrary(hAssistantTools);
}
return 0;
}
```

### 2.编译源文件`fuzz_program.cpp`得到`fuzz_program.exe`

```
g++ -g -o fuzz\_program.exe fuzz\_program.cpp
```

## 五、准备fuzz

1. 准备在目录`D:\\_\_workspace\_\_\fuzz\test\_winafl\thunder\_torrent`下进行fuzz；
2. 将`AssistantTools.dll`拷贝到目录下；

`AssistantTools.dll`来自`D:\softwares\Thunder Network\Thunder\Program\resources\bin\SDK\AssistantTools.dll`

3. 由于`AssistantTools.dll`依赖`P2PBase.dll`，所以也需要将`P2PBase.dll`拷贝到目录下；

`P2PBase.dll`来自`D:\softwares\Thunder Network\Thunder\Program\resources\bin\SDK\P2PBase.dll`

4. 将`winafl.dll`拷贝到目录下；

`winafl.dll`来自`D:\\_\_workspace\_\_\fuzz\winafl\build\_Win32\bin\Release\winafl.dll`

5. 准备一个语料库目录`samples`，里面塞满各种地方找来的torrent文件；
6. 最后目录下如图所示；

![图片描述](upload/attach/202102/825829\_H42XYQ9JHDF6PRF.png)

## 六、生成覆盖率文件

1. 通过`dynamorio`工具对`fuzz_program.exe`在解析torrent文件时的执行流程进行污点跟踪，得到二进制的覆盖率文件，执行命令如下；

```
"D:\\_\_workspace\_\_\fuzz\dynamorio\build\_Win32\bin32\drrun.exe" -t drcov -- "fuzz\_program.exe" "D:\\_\_workspace\_\_\fuzz\test\_winafl\thunder\_torrent\samples\test.torrent"
```

2. 之后在当前目录下会看到生成的`drcov`前缀的log文件；

![图片描述](upload/attach/202102/825829\_GGGPWBDDPXZZJXW.png)

3. 使用32位`ida`打开`AssistantTools.dll`，使用`ida`插件`lighthouse`打开刚才生成的log文件；

![图片描述](upload/attach/202102/825829\_Z794ED5W5RCSNPW.png)

4. 简单检查一下目标函数`XL_ParseTorrentFileW`的代码覆盖情况，绿色的为覆盖的部分；

![图片描述](upload/attach/202102/825829\_8YC38RWJUE27RVK.png)

## 七、对语料库samples最小化到minset目录下

```
"C:\Python27\python.exe" "D:\\_\_workspace\_\_\fuzz\winafl\winafl-cmin.py" --working-dir "D:\\_\_workspace\_\_\fuzz\winafl\build\_Win32\bin\Release" -D "D:\\_\_workspace\_\_\fuzz\dynamorio\build\_Win32\bin32" -t 9000 -i "D:\\_\_workspace\_\_\fuzz\test\_winafl\thunder\_torrent\samples" -o "D:\\_\_workspace\_\_\fuzz\test\_winafl\thunder\_torrent\minset" -coverage\_module AssistantTools.dll -coverage\_module P2PBase.dll -target\_module fuzz\_program.exe -target\_method fuzz\_method -nargs 1 -- "D:\\_\_workspace\_\_\fuzz\test\_winafl\thunder\_torrent\fuzz\_program.exe" @@
```

![图片描述](upload/attach/202102/825829\_DGB3MX2ZPP9CDTP.png)

可以看到，原本`samples`目录中有92个测试样本，最小化到`minset`目录后得到13个测试样本；

## 八、获取fuzz_program.exe在运行时的模块加载状况、文件打开状况、网络连接状况

```
"D:\\_\_workspace\_\_\fuzz\dynamorio\build\_Win32\bin32\drrun.exe" -c winafl.dll -debug -coverage\_module AssistantTools.dll -coverage\_module P2PBase.dll -target\_module fuzz\_program.exe -target\_method fuzz\_method -fuzz\_iterations 10 -nargs 1 -- "D:\\_\_workspace\_\_\fuzz\test\_winafl\thunder\_torrent\fuzz\_program.exe" "D:\\_\_workspace\_\_\fuzz\test\_winafl\thunder\_torrent\samples\test.torrent"
```

之后生成`afl`前缀的log文件，运行`afl-fuzz.exe`前设置的`coverage_module`参数和`target_module`参数的值只能从log文件中列出的模块里获取。

## 九、开始fuzz

计划使用一个`master`，两个`slave`分别为`slave01`和`slave02`；

### 1、master

```
"D:\\_\_workspace\_\_\fuzz\winafl\build\_Win32\bin\Release\afl-fuzz.exe" -i "D:\\_\_workspace\_\_\fuzz\test\_winafl\thunder\_torrent\minset" -M master -o "D:\\_\_workspace\_\_\fuzz\test\_winafl\thunder\_torrent\result" -D "D:\\_\_workspace\_\_\fuzz\dynamorio\build\_Win32\bin32" -I 100000+ -t 9000 -- -coverage\_module AssistantTools.dll -coverage\_module P2PBase.dll -target\_module fuzz\_program.exe -target\_method fuzz\_method -fuzz\_iterations 5000 -nargs 1 -- "D:\\_\_workspace\_\_\fuzz\test\_winafl\thunder\_torrent\fuzz\_program.exe" @@
```

![图片描述](upload/attach/202102/825829\_EMACT96JKK7BGJ5.png)

### 2、slave01

```
"D:\\_\_workspace\_\_\fuzz\winafl\build\_Win32\bin\Release\afl-fuzz.exe" -i "D:\\_\_workspace\_\_\fuzz\test\_winafl\thunder\_torrent\minset" -S slave01 -o "D:\\_\_workspace\_\_\fuzz\test\_winafl\thunder\_torrent\result" -D "D:\\_\_workspace\_\_\fuzz\dynamorio\build\_Win32\bin32" -I 100000+ -t 9000 -- -coverage\_module AssistantTools.dll -coverage\_module P2PBase.dll -target\_module fuzz\_program.exe -target\_method fuzz\_method -fuzz\_iterations 5000 -nargs 1 -- "D:\\_\_workspace\_\_\fuzz\test\_winafl\thunder\_torrent\fuzz\_program.exe" @@
```

![图片描述](upload/attach/202102/825829\_CPDZCBAVPTZRA8M.png)

### 3、slave02

```
"D:\\_\_workspace\_\_\fuzz\winafl\build\_Win32\bin\Release\afl-fuzz.exe" -i "D:\\_\_workspace\_\_\fuzz\test\_winafl\thunder\_torrent\minset" -S slave02 -o "D:\\_\_workspace\_\_\fuzz\test\_winafl\thunder\_torrent\result" -D "D:\\_\_workspace\_\_\fuzz\dynamorio\build\_Win32\bin32" -I 100000+ -t 9000 -- -coverage\_module AssistantTools.dll -coverage\_module P2PBase.dll -target\_module fuzz\_program.exe -target\_method fuzz\_method -fuzz\_iterations 5000 -nargs 1 -- "D:\\_\_workspace\_\_\fuzz\test\_winafl\thunder\_torrent\fuzz\_program.exe" @@
```

![图片描述](upload/attach/202102/825829\_WHM6P2YG9CF8DC8.png)

## 十、中断fuzz过程

`Ctrl+C`

## 十一、恢复fuzz

恢复fuzz只需要将原命令的`-i`参数修改为`-`即可，其他参数不变；

### 1、master

```
"D:\\_\_workspace\_\_\fuzz\winafl\build\_Win32\bin\Release\afl-fuzz.exe" -i - -M master -o "D:\\_\_workspace\_\_\fuzz\test\_winafl\thunder\_torrent\result" -D "D:\\_\_workspace\_\_\fuzz\dynamorio\build\_Win32\bin32" -I 100000+ -t 9000 -- -coverage\_module AssistantTools.dll -coverage\_module P2PBase.dll -target\_module fuzz\_program.exe -target\_method fuzz\_method -fuzz\_iterations 5000 -nargs 1 -- "D:\\_\_workspace\_\_\fuzz\test\_winafl\thunder\_torrent\fuzz\_program.exe" @@
```

### 2、slave01
