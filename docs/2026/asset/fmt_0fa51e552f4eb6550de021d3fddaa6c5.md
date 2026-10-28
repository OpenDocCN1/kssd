
```
(Get-Command Invoke-Expression).ImplementingType.AssemblyQualifiedName
```

![图片描述](https://bbs.kanxue.com/upload/attach/201908/830989_RT34D77FSRXNRTE.png)

我们用`dnspy`打开GAC程序集，并定位到该模块

![图片描述](https://bbs.kanxue.com/upload/attach/201908/830989_X8FXVZNFRFMFQYN.png)

定位到`Microsoft.Powershell.Commands`包下的`InvokeExpressionCommand`类，发现该类只有一个函数

![图片描述](https://bbs.kanxue.com/upload/attach/201908/830989_SQ5Y62XE3VKKUDH.png)

验证是不是我们要Hook的函数，下个断点，调试程序`powershell_ise.exe`，并执行`iex "get-process"`,发现确实是我们要Hook的代码

![图片描述](https://bbs.kanxue.com/upload/attach/201908/830989_SXCDNZGT4M3HJ26.png)

我们在这里尝试修改代码是可以改的，但是却不能让程序执行我们的代码，这个模块所在的位置在用户层是没有权限访问的，或者说`exploer.exe`（资源管理器）是没有权限访问到这个目录的

![图片描述](https://bbs.kanxue.com/upload/attach/201908/830989_6EM9RC44P5UB35X.png)

在修改完Hook代码后，这里有两种方式替换掉目标dll:

第一种：用ARK工具，例如`PcHunter`等工具可以访问到该目录，并且可以删除掉。

![图片描述](https://bbs.kanxue.com/upload/attach/201908/830989_TV4DKNFETCK3QHZ.png)

然后用`cmd`命令，复制到该目录（**缺陷**:`powershell_ise`程序在被调试加载和单独运行的时候，加载的模块并不是通一个，然而另外一个模块，用`cmd`复制过去也是没有用的）

```
move C:\Users\15pb-win7\Desktop\Microsoft.PowerShell.Commands.Utility.dll  C:\Windows\assembly\GAC_MSIL\Microsoft.PowerShell.Commands.Utility\1.0.0.0__31bf3856ad364e35\
```

第二种，自己编写`patch`程序（驱动模块，因为需要system权限，我们可以替换`cmd`的`token`,也可以自己复制替换）

## patch程序的编写

### 0x01--原始DLL的修改

dll模块的Hook代码，有过C#开发经验的，编写代码还是比较简单的。

我们在这里不改变程序的流程，只把参数记录下来，并写文件到C盘。改后如下：

![图片描述](https://bbs.kanxue.com/upload/attach/201908/830989_ETKVRUXBK3D773F.png)

修改后将代码保存到桌面，原始路径你是没有权限保存的。

### 0x02--3环程序的编写

之前有说过，我们直接运行`powershell_ise`和在调试器里运行，其加载的模块是不一样，如下图：

调试时要hook的模块：

![图片描述](https://bbs.kanxue.com/upload/attach/201908/830989_5SPGDGQSA6Y5CSD.png)

直接运行加载的模块：

![图片描述](https://bbs.kanxue.com/upload/attach/201908/830989_RMAEPC6YXV29PT6.png)

经过分析，加载的模块虽然大小不一样，但是包是一样的，起码我们要改的代码处是一样的。

程序正常运行时，如果`Microsoft.PowerShell.Commands.Utility.ni.dll`不存在则加载`Microsoft.PowerShell.Commands.Utility.dll`

我们的`patch`程序，要做的事有如下几件：

- 删掉`Microsoft.PowerShell.Commands.Utility.dll`
- 删掉`Microsoft.PowerShell.Commands.Utility.ni.dll`
- 复制我们修改过dll，到`Microsoft.PowerShell.Commands.Utility.dll`所在目录下

注：必须要先删除，直接覆盖是不行的

正常运行时的路径如下：

```
C:\Windows\assembly\Nativeimages_v2.0.50727_32\Microsoft.PowerShel#\4bdde288f147e3b3f2c090ecdf704e6d\Microsoft.PowerShell.ConsoleHost.ni.dll
```

`C:\Windows\assembly\Nativeimages_v2.0.50727_32\Microsoft.PowerShel#\`不变，后边是在变化的，我们要去动态获取。

获取代码(对该目录进行递归，找到文件，保存当前目录路径):

```
void EnumFile(TCHAR *szPath, TCHAR* szFileName)
{
    TCHAR szTempPath[MAX_PATH] = {};
    wsprintf(szTempPath, L"%s\\*", szPath);
    WIN32_FIND_DATA FindData = {};
    TCHAR szNextPath[MAX_PATH] = {};
    TCHAR szOutputPant[MAX_PATH] = {};
    HANDLE hFind = FindFirstFile(szTempPath, &FindData);
    do
    {
        if (FindData.dwFileAttributes & FILE_ATTRIBUTE_DIRECTORY)//输出目录
        {
            if (wcscmp(FindData.cFileName, L".") == 0 || wcscmp(FindData.cFileName, L"..") == 0)
            {
                //如果是这俩目录，就什么都不干
            }
            else
            {
                wsprintf(szNextPath, L"%s\\%s", szPath, FindData.cFileName);
                EnumFile(szNextPath, szFileName);
            }
        }
        else
        {
            //如果不是目录，就查看这个玩意是不是我们的目标文件
            if (wcscmp(szFileName, FindData.cFileName) == NULL)
            {
                wsprintf(szDestPath, L"\\??\\%s\\%s", szPath, FindData.cFileName);
                //是就输出
                printf("%ws\n", szDestPath);
            }
        }
    } while (FindNextFile(hFind, &FindData));
    FindClose(hFind);
}
```

将获取的路径写到本地（也可以不用写，把消息发送到0环也行）

![图片描述](https://bbs.kanxue.com/upload/attach/201908/830989_7GFVUFUKECEY4WM.png)

### 0x03--驱动程序的编写

3环的程序，就是为了获取路径

那么0环的程序，自然就是要读取路径，删除，复制（比较简单，并没有什么技术含量）

![图片描述](https://bbs.kanxue.com/upload/attach/201908/830989_DZFVW2MG7KA6HAF.png)

在0环是没有删除文件和读取文件的，需要自己封装。

驱动的所有代码如下：
