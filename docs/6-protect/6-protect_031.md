
# 想自己实现一个 LoadLibrary，请教

> 原文：[`www.pediy.com/kssd/pediy10/75365.html`](https://www.pediy.com/kssd/pediy10/75365.html)

【求助】想自己实现一个 `LoadLibrary`，请教

流程如下：

先把文件按节表映射入内存，处理重定位，`LoadLibrary` 导入表中的 DLL 并完成 IAT。

是不是这样就可以了？？

谢谢楼上几位

因为一些特殊的要求，所以要兼容大部分的系统 DLL，本来用一种简单的方法已经实现的差不多，没想到在 `vista home basic` 居然出问题，`vista ultimate` 却没问题。。。这年头操作系统真 TM 太多了。还在想其他办法，比如先用 `LoadLibraryEx` 带 `DONT_RESOLVE_DLL_REFERENCES` 实现，再自己进行初始化，如果再没办法只好自己实现 `LOADLIBRARY` 了，

`TLS` 倒是要考虑进去的。还有加载完成后调用 `DLLMAIN`。

TO：笨笨雄

还需要初始化啥东西？

我昨天在网上找了篇文章，说是把一个 DLL 模块进行隐藏的方法，先把内容复制到一块内存，再 `FreeLibrary` 掉，然后复制回来，这样就不会显示在模块列表中了，我的要求和这个也很类似，问题在于我的目标 DLL 模块是无法用 `FreeLibrary` 卸载的，即使用 `NtUnmapViewOfSection` 卸载掉，那内存也无法再使用。。。。估计在内核中有什么记号还在吧

```c
const char* DllName = "fuckme.dll";

void FuckMe()
{
    void *PEB = NULL;
    void *Ldr = NULL;
    _LIST_ENTRY *Flink = NULL;
    _LIST_ENTRY *p = NULL;
    BYTE    *BaseAddress = NULL;
    BYTE    *FullDllName = NULL;
    __asm
    {
        mov eax,fs:[0x30]
        mov PEB,eax
    }
    Ldr = *( ( void ** )( ( unsigned char * )PEB+0x0c ) );
    Flink = (_LIST_ENTRY*)*( ( void ** )( ( unsigned char * )Ldr+ 0x0c ) );
    p = Flink;
    do
    {
        BaseAddress = *( ( BYTE ** )( ( unsigned char * )p+ 0x18 ) );
        FullDllName = *( ( BYTE ** )( ( unsigned char * )p+ 0x28 ) );
        LPSTR strFullDllName;
        UnicodeToAnsi((LPCOLESTR)FullDllName,&strFullDllName);
        if( strFullDllName )
        {
            if( strstr(strFullDllName,DllName) )
            {
                *(LPDWORD)((LPBYTE)p + 0x38) = 1;
            }
        }
        CO_SAFE_DELETE(strFullDllName);
        p = p->Flink;
    }
    while ( Flink != p );
}

FuckMe();

FreeLibrary(DllName);
```
