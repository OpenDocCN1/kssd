
3） 触发反调试后，对代码段进行异或处理。被修改的代码段执行后一定会崩掉。

之所以设计以上三个流程。是为了让破解者产生一种错觉。

类似于“这次程序没退出，这货的反调试应该被我清干净了”。

## 可用反调试1

这个反调试是直接检测多款 od 的窗口风格，使用`GetWindowLongA`获取窗口的`Style`进行对比。

这里采集了几款主流调试器：

![](https://bbs.kanxue.com/upload/attach/202001/93_K3UADYSDMPJ7VEN.jpg)

可以看到上面的 od 窗口风格是 0x16CF0000

![](https://bbs.kanxue.com/upload/attach/202001/93_6XQCEJA6DWSVF5E.jpg)

可以看到上面的窗口风格是 0x17CF0000

![](https://bbs.kanxue.com/upload/attach/202001/93_Z6PHCKFH7PRSGBH.jpg)

x32dbg 这里是 0x97cf0000

od 的窗口风格大概差不多，测试了多款 OD 后，选取了以上 3 个固定值进行对比。

代码如下：

```
#define NUM_STYLE_OD1 0x17CF0000

#define NUM_STYLE_OD2 0x16cF0000

#define NUM_STYLE_X32 0x97cf0000

bool Anti::wnd_long_debug()
{
    HWND h_debug_wnd = 0;
    h_debug_wnd = GetForegroundWindow();

    if (0 == h_debug_wnd)
    {
        return false;
    }

    LONG l_style = GetWindowLongA(h_debug_wnd, GWL_STYLE);

    if (NUM_STYLE_OD1 == l_style || NUM_STYLE_OD2 == l_style 
     || NUM_STYLE_X32 == l_style)
    {
        return true;
    }

    return false;
}
```

经过实测，x32dbg 只有在全屏的时候能够检测到，别的 OD 调试器检测率很高。

![](https://bbs.kanxue.com/upload/attach/202001/93_8NZ5TDSHSDSTH7G.jpg)

![](https://bbs.kanxue.com/upload/attach/202001/93_F9DHJCR2NBX5KA9.jpg)

## 可用反调试2

这里是检测`PEB`的两个标志位。

正常运行时，两个标志位的值都是 1，但是调试状态下会被修改。

x32dbg 调试运行

![](https://bbs.kanxue.com/upload/attach/202001/93_NG24JECGHYFN6HV.jpg)

普通 od 调试运行

![](https://bbs.kanxue.com/upload/attach/202001/93_JWSJQJR72W6FVWU.jpg)

以下代码应该是“原创”，因为我并没有在别的地方看到类似的反调试。

（当然也有可能各位大佬早就研究出来没有公布，或者公布了我不知道）

```
bool Anti::flag_debug()
{
    bool b_ret = false;

    __asm
    {
        mov eax, fs:[0x30]
        mov eax, dword ptr[eax + 0x10]

        cmp byte ptr[eax + 0x68], 0x81
        je anti_addr

        cmp byte ptr[eax + 0x68], 0x0
        je anti_addr

        cmp byte ptr[eax + 0x6c], 0xa
        je anti_addr

        cmp byte ptr[eax + 0x6c], 0x0
        je anti_addr

    ok_addr :
        mov b_ret, 0
        jmp ret_addr

    anti_addr :
        mov b_ret, 1

    ret_addr:
        mov eax, 1
    }

    return b_ret;
}
```

主流调试器在上面的反调试中，都会被检测到。

![](https://bbs.kanxue.com/upload/attach/202001/93_EQP96SFQFYQ6UAS.jpg)

![](https://bbs.kanxue.com/upload/attach/202001/93_ERJ6SPN8KWMX786.jpg)

以上反调试相关，实名感谢一个高中生大佬@狐白小刺客的思路和指点。

## 耦合相关

我们写代码最好做到高内聚、低耦合。但是写壳子就不一样了，最好做到高耦合性。也就是说，外壳与原程序做到紧密联结，脱完这个壳子，程序就不能跑了。

当前的壳子实现了两种比较简单的方法，有些没实现但是很好用的方法我也会写出来。

### 1) 互斥体检测实现耦合

具体思路：外壳 shellcode 创建互斥体对象，源程序进行检测这个内核对象是否存在，不存在则表示被脱壳。

```
//这里在 shellcode处创建了一个互斥体对象
void coup_event(PFN_CreateMutexA pfn_cre_mutex)
{
    char sz_event_name[] = { 'l', 'y', 'd', '\0' };
    pfn_cre_mutex(NULL, FALSE, sz_event_name);  //创建互斥体
}
```

源程序的检测代码和检测到的处理修改代码：

```
//源程序中的检测互斥体是否存在的代码 
//true 存在互斥体 false壳子被脱
bool Coup::coup_mutex_func()
{
    HANDLE h_mutex = 0;
    h_mutex = CreateMutexA(NULL, TRUE, STR_EVENT_NAME);  //创建互斥体

    DWORD dw_error_code = GetLastError();

    //如果存在 不做处理
    if (ERROR_ALREADY_EXISTS == dw_error_code)
    {
        CloseHandle(h_mutex);
        return true;
    }

    return false;
}

//检测内核对象是否存在的耦合性处理代码  不存在则修改数据
//---------耦合相关---------------
bool b_ret_mutex = Coup::coup_mutex_func();

if (!b_ret_mutex)
{
    if (0 != g_p_diy)
    {
        //被拖壳修改数据
        //puts("壳子已经被脱, 修改数据中");
        g_p_diy->Diy_Upd_Data();
    }
}
//---------耦合相关---------------
```

### 2) 独占方式创建文件实现耦合性检测

实现思路：外壳 shelloce 端创建一个独占形式打开的文件，源程序里面能正常打开则表示壳子被脱，此刻就可以修改数据或者退出进程。

```
//外壳shelloce端创建一个独占形式打开的文件
void coup_file(PFN_CreateFileA pfn_createfileA, PFN_DeleteFileA pfn_DeleteFileA)
{
    HANDLE h_file = 0;
    char sz_file_path[] = { '-','\0' };

    //存在就删除
    pfn_DeleteFileA(sz_file_path);

    //以独占打开一个文件 不存在则创建
    h_file = pfn_createfileA(sz_file_path, // open One.txt
        0x80000000L,                       // open for reading  GENERIC_READ
        0,                                 // do not share
        0,                                 // no security
        1,                                 // 不存在则创建
        0x00000080,                        // normal file  0x00000080  FILE_ATTRIBUTE_NORMAL
        NULL);

    DWORD dw_ww = 0x226688;
}
```

源程序的检测代码和检测到后的处理代码：

```
//true 存在文件 false壳子被脱 检测代码
bool Coup::coup_file_func()
{
    HANDLE h_file = 0;

    h_file = CreateFileA("-",     // open 0.txt
        GENERIC_READ,             // open for reading
        0,                        // do not share
        NULL,                     // no security
        OPEN_EXISTING,            // existing file only
        FILE_ATTRIBUTE_NORMAL,    // normal file
        NULL);

    //DWORD dw_xx = 0x221166;

    DWORD dw_error_code = GetLastError();

    //如果成功打开 壳子已经被脱
    if (INVALID_HANDLE_VALUE != h_file)
    {
        return false;
    }

    //如果报共享文件冲突 表示壳子存在
    if (ERROR_SHARING_VIOLATION == dw_error_code)
    {
        DeleteFileA("-");
        return true;
    }

    return false;
}

//检测耦合性的处理相关代码
//---------耦合相关---------------
bool b_ret_file = Coup::coup_file_func();

if (!b_ret_file)
{
    if (0 != g_p_diy)
    {
        //被拖壳修改数据
        //puts("壳子已经被脱, 修改数据中");
        g_p_diy->Diy_Upd_Data();
    }
}
//---------耦合相关---------------
```

以上是两种比较简单的思路。有一些思路有同样的作用但是并未实现，比如：

- 外壳 shellcode 往本机的端口发送一个数据包。
- 源程序在代码里面接收数据包。如果没有收到，就表示壳子被脱了。

## 中断门相关

为了防止 API 下断点，可以用 windows 的中断门实现 API。这里如果要用到中断，就必须绑定平台。最开始设计壳子的时候，设定要运行的环境是 w7 32 位，所以这里 API 的中断门是以 w7 平台为目标平台。

ida 打开 w7 32 位的`kernel32.dll`，找到`ExitProcess`：

![](https://bbs.kanxue.com/upload/attach/202001/93_H9WFQB45R4V7BV3.jpg)

发现调用了另一个函数`RtlExitUserProcess`，这里 ida 打开 w7 32 位`ntdll.dll`：

发现调用了`ZwTerminateProcess`：

![](https://bbs.kanxue.com/upload/attach/202001/93_DKPPQUWHDURF3MK.jpg)

查看`ZwTerminateProcess`函数：

![](https://bbs.kanxue.com/upload/attach/202001/93_EBV78TJF4RRKRYU.jpg)

所以中断门实现`ExitProcess`的代码如下：

```
__declspec(naked) void exit_proc()
{
    __asm
    {
        push   0         //退出码
        push - 1         //表示退出当前进程
        mov eax, 0x172   //中断号
        mov edx, esp
        int 0x2e
        add esp, 8
        ret
    }
}
```

## smc相关

加壳程序只是以一个固定值做了代码段的加密，也就是当前解密的次数 `* 2` 进行了异或；shellcode 端也以一样的 key 进行了异或。

加壳器端代码：

```
int Cmps_Shell::enc_code_seg(PBYTE p_code_base, int n_code_size, PBYTE p_next_seg_buf)
{
    BYTE c_key = 0;
    int n_idx = 0;
    int n_dec_oft = 0;
    int n_cur_enc_size = n_code_size;
    int n_remain_size = n_code_size;

    while (true)
    {
        n_cur_enc_size = n_remain_size;

        //如果大于0x2000 一次性加密  0x2000 以外所有数据
        //比如代码段 0x9000 一次性加密0x7000
        if (n_cur_enc_size > NUM_FIRST_DEC_VALUE)
        {
            n_cur_enc_size -= NUM_FIRST_DEC_VALUE;
        }

        //>100 <2000 剩下的每次加密一半
        else if (n_cur_enc_size > NUM_NEXT_DEC_VALUE)
        {
            n_cur_enc_size = n_remain_size / 2;
        }

        c_key = enc_get_key(n_idx, p_next_seg_buf);
        this->enc_code_exec(p_code_base + n_dec_oft,
                            n_cur_enc_size,
                            c_key);

        n_remain_size -= n_cur_enc_size;
        n_dec_oft += n_cur_enc_size;

        if (n_remain_size <= 0)
        {
            break;
        }

        n_idx++;
    }

    return 0;
}
```

shellcode 端解密代码
