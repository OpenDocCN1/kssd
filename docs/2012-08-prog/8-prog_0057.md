
# 自定义 DLL 入口点 dllmain()的第 2 个参数数值传递从而达到隐藏导出接口

> 原文：[`www.pediy.com/kssd/pediy12/143308.html`](https://www.pediy.com/kssd/pediy12/143308.html)

**[作者]**

网名:猪头三

个人网站:[`www.x86asm.com`](http://www.x86asm.com)

**[序言]**

很久不研究另类的编码技术了,不知怎么的这几天突然心血来潮,突然想分享一个有趣的技术给大家.

**[技术作用]**

我们知道 dll 的主要作用是提供接口别人使用,按照正规的方式需要把函数名导出出来,但这对于有安全瘾的人来说,但有时我们需要更加安全不想把函数名导出.下面我提供一个技术.

**[声明]**

为了避免不必要的争吵,我把这内容定位在编程技巧，而不是定位在技术讨论.

**[原理]**

DLL 入口点的声明为:

```
BOOL APIENTRY DllMain( HMODULE hModule,
                       DWORD  ul_reason_for_call,
                       LPVOID lpReserved
                     )
```

第 2 个参数按照 MSDN 文档解释只能接收他规定的数值比如:

- `DLL_PROCESS_ATTACH`
- `DLL_THREAD_ATTACH`
- `DLL_THREAD_DETACH`
- `DLL_PROCESS_DETACH`

但有时想接受更多扩展定义,比如 `100` `101` `102` 这些自定义数值怎么办呢?

如果接收更多扩展定义,那我们写接口的时候根本不用导出函数名了,只要让调用者知道这些扩展定义,并对 DLL 发送这些定义数值,DLL 内部可以根据这些 ID 进行识别做对应的功能处理.

这样是不是相对更加安全呢？^\_^

下面我们继续讨论原理.......

要让 DLL 能接收到这些扩展定义,只要让调用者找到 DLL 的入口点,并调用就 OK 了.下面我们开始实战.

**[基础理论]**

1. DLL 模块的使用原理
2. PE 结构的理解
3. C/C++语言
4. VS 开发工具的使用

**[DLL 内部源码]**

```
BOOL APIENTRY DllMain( HMODULE hModule,
                       DWORD  ul_reason_for_call,
                       LPVOID lpReserved
                     )
{
    switch (ul_reason_for_call)
    {
    case DLL_PROCESS_ATTACH:
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
    case DLL_PROCESS_DETACH:
        break;
    case 100: // 扩增定义
        {
            printf("自定义已经传递进来了 ID: 100 \n") ;
        }
    break;
    case 101: // 扩增定义
        {
            printf("自定义已经传递进来了 ID: 101\n") ;
        }
        break;
    case 102: // 扩增定义
        {
            printf("自定义已经传递进来了 ID: 101\n") ;
        }
        break;
    }
    return TRUE;
}
```

**[调用 DLL 源码]**

```
int _tmain(int argc, _TCHAR* argv[])
{
    DWORD dword_Error ;
    // PE 文件头
    PIMAGE_DOS_HEADER pstruct_ImageDOSHeaders = NULL ;
    PIMAGE_NT_HEADERS pstruct_pImageNTHeaders = NULL ;
    // 加载目标 DLL
    HINSTANCE handle_Dll = ::LoadLibrary("Test_Dll.dll") ;

    if (handle_Dll == NULL)
    {
        return 0 ;
    }

    // 校验 PE 头是否合法
    pstruct_ImageDOSHeaders = (PIMAGE_DOS_HEADER)handle_Dll;
    if (pstruct_ImageDOSHeaders->e_magic != IMAGE_DOS_SIGNATURE)
        return 0 ;

    // 定位 PE 的 NT 头并校验
    pstruct_pImageNTHeaders = (PIMAGE_NT_HEADERS)((PBYTE)pstruct_ImageDOSHeaders + pstruct_ImageDOSHeaders->e_lfanew);
    if (pstruct_pImageNTHeaders->Signature != IMAGE_NT_SIGNATURE)
        return 0 ;

    // 定义 DLLMAIN 入口函数并获取入口地址
    BOOL (APIENTRY *DllMain)( HMODULE hModule,
                              DWORD  ul_reason_for_call,
                              LPVOID lpReserved);

    DllMain = (BOOL (APIENTRY *)( HMODULE, DWORD, LPVOID))(((PBYTE)handle_Dll) + pstruct_pImageNTHeaders->OptionalHeader.AddressOfEntryPoint);

    // 开始发送扩展定义
    DllMain(handle_Dll, 100, &dword_Error );
    DllMain(handle_Dll, 101, &dword_Error );
    DllMain(handle_Dll, 101, &dword_Error );

    return 0;
}
```

**[结束]**

希望这个编程技巧对大家有用,感谢大家对这个技巧的支持
