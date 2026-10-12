
# 《游戏修改器 DIY 之三》——用 C#读写游戏进程内存

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-127.htm`](https://www.pediy.com/kssd/pediy09/pediy09-127.htm)

> 《游戏修改器 DIY 之三》  
> ——用 C#读写游戏进程内存  
>   
> 天津 赵春生

这是《游戏修改器 DIY》系列的第三篇，通过前两篇拙作相信大家对游戏修改器的制作有了初步的了解，前两篇使用的编程工具是“Microsoft Visual C++ 6.0”，而这次将使用“Microsoft Visual C# 2005 Express Edition”。在 C#中使用 API 可不像在 C/C++那么简单，因为对我来说，C#还很陌生……

本篇不再赘述游戏修改器的原理了，如果对此不了解的朋友可先看看我写的前两篇拙作或其他相关资料。

## 一：在 C#中使用 API。

首先要导入一个 namespace，以便简化后面的代码输入：

```
using System.Runtime.InteropServices;
```

其次要使用 `DllImport` 引入 API 函数，如：

```
[DllImport("user32.dll", CharSet = CharSet.Auto)]
public static extern int MessageBox(IntPtr hWnd, String text, String caption, uint type);
```

现在就可以使用了，如：

```
MessageBox(new IntPtr(0), "Hello World!", "Hello Dialog", 0);
```

为了使用方便，我制作了一个简单的类：

```
class ProcessMemoryWorkApi
{
    //HANDLE OpenProcess(
    //    DWORD dwDesiredAccess,// access flag
    //    BOOL bInheritHandle, // handle inheritance flag
    //    DWORD dwProcessId  // process identifier
    //    );
    [DllImport("kernel32.dll")]
    public static extern
        IntPtr OpenProcess(UInt32 dwDesiredAccess, Int32 bInheritHandle, UInt32 dwProcessId);

    //BOOL CloseHandle(
    //    hObject  // handle to object to close
    //    );
    [DllImport("kernel32.dll")]
    public static extern
        Int32 CloseHandle(IntPtr hObject);

    //BOOL WriteProcessMemory(
    //    HANDLE hProcess, // handle to process whose memory is written to
    //    LPVOID lpBaseAddress, // address to start writing to
    //    LPVOID lpBuffer, // pointer to buffer to write data to
    //    DWORD nSize, // number of bytes to write
    //    LPDWORD lpNumberOfBytesWritten  // actual number of bytes written
    //    );
    [DllImport("kernel32.dll")]
    public static extern
        Int32 WriteProcessMemory(IntPtr hProcess, IntPtr lpBaseAddress, uint[] lpBuffer, UInt32 nSize, IntPtr lpNumberOfBytesWritten);

    //BOOL ReadProcessMemory(
    //    HANDLE hProcess, // handle of the process whose memory is read
    //    LPCVOID lpBaseAddress, // address to start reading
    //    LPVOID lpBuffer, // address of buffer to place read data
    //    DWORD nSize, // number of bytes to read
    //    LPDWORD lpNumberOfBytesRead  // address of number of bytes read
    //    );
    [DllImport("kernel32.dll")]
    public static extern
        Int32 ReadProcessMemory(IntPtr hProcess, IntPtr lpBaseAddress, uint[] lpBuffer, UInt32 nSize, IntPtr lpNumberOfBytesRead);
}
```

## 二：用 C#制作游戏修改器。

游戏是大家熟悉的“三维弹球”，这是 WinXP 自带的一款游戏，目的是修改她的分数，选这个游戏的原因是因为大家都能很容易地得到她，而且修改起来也很方便：取 `0x1025040` 中的值，然后与 `0x52` 相加，即可得到存放游戏分数的地址。

单击“Go”按钮，可显示当前分数，并将分数改为：“999600000”，关键代码如下：

```
private void button_go_Click(object sender, EventArgs e)
    {
        IntPtr PinballHandle;
        IntPtr PinballScoreAddressPointer = (IntPtr)0x1025040;  //内存 0x1025040 中的值+0x52->PinballScoreAddress
        uint[] PinballScoreAddress = new uint[1];               //游戏分数的内存地址
        uint[] PinballScoreWrite = new uint[] { 0x3B94AF80 };   //将分数改为:999600000
        uint[] PinballScoreRead = new uint[1];                  //用来存放当前分数

        //"3D Pinball"在“Windows 任务管理器”中显示为"PINBALL.EXE"，但作为
        //GetProcessesByName 的参数使用时并不能包含扩展名".EXE"。
        System.Diagnostics.Process[] PinballProcessID
            = System.Diagnostics.Process.GetProcessesByName("PINBALL"); //根据进程名获取进程 ID

        if (PinballProcessID.Length != 0)
        {
            PinballHandle = ProcessMemoryWorkApi.OpenProcess(0x1F0FFF, 0, (UInt32)PinballProcessID[0].Id);  //PROCESS_ALL_ACCESS=0x1F0FFF
            ProcessMemoryWorkApi.ReadProcessMemory(PinballHandle, PinballScoreAddressPointer, PinballScoreAddress, 4, (IntPtr)0);//通过指针得到 PinballScoreAddress
            PinballScoreAddress[0] = PinballScoreAddress[0] + 0x52; //PinballScoreAddress+0x52 才是游戏分数的内存地址
            ProcessMemoryWorkApi.ReadProcessMemory(PinballHandle, (IntPtr)PinballScoreAddress[0], PinballScoreRead, 4, (IntPtr)0);     //当前游戏分数
            MessageBox.Show("当前分数为： " + PinballScoreRead[0].ToString() + "\\n" + "分数即将被修改成： 999600000", "ProcessMemoryWork_Demo");
            ProcessMemoryWorkApi.WriteProcessMemory(PinballHandle, (IntPtr)PinballScoreAddress[0], PinballScoreWrite, 4, (IntPtr)0);    //修改游戏分数
            ProcessMemoryWorkApi.CloseHandle(PinballHandle);
        }
        else
            MessageBox.Show("\"3D Pinball\" 还没有运行吧？", "ProcessMemoryWork_Demo");

    }
```

## 三：相关提示。

刚刚接触 C#不久，对 OOP 也一知半解，写此文只为抛砖引玉，谢谢！

本程序代码测试环境：

WinXP (Professional SP2)  
Microsoft Visual Studio 2005 (Installed Edition: C# Express)  
Version `8.0.50727.762` (SP.050727-7600)  
Microsoft .NET Framework  
Version `2.0.50727`

《游戏修改器 DIY》/《游戏修改器 DIY 之二》可通过搜索引擎或访问我的 Blog 获得。

Blog：

[`timw.yeah.net`](http://timw.yeah.net)  
[`timw.126.com`](http://timw.126.com)

20:35 2007-04-27
