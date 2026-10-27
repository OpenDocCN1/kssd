
```
                lstrcpynA( buf, pszModule + 1, dwSize );
            }
            else
            {
                lstrcpynA( buf, pList->smi[i].ImageName, dwSize );
            }
            return TRUE;
        }
    }
    return FALSE;
}
```

## 详细枚举系统服务项

到现在为止，还遗留有一个问题，就是获得服务号对应的服务函数名。比如XP下`0x7A`对应着`NtOpenProcess`，但是到2000下，`NtOpenProcess`就改为`0x6A`了。

有一个好消息一个坏消息，你先听哪个？

什么坏消息？

Windows并没有给我们开放这样现成的函数，所有的工作都需要我们自己来做。

那好消息呢？

牛粪有的是。

坏了，串词儿了。好消息是我们可以通过枚举`ntdll.dll`的导出函数来间接枚举SSDT所有表项所对应的函数，因为所有的内核服务函数对应于`ntdll.dll`的同名函数都是这样开头的：

```
mov eax, <ServiceIndex>
```

对应的机器码为：

```
B8 <ServiceIndex>
```

再说一遍：非常幸运，仅就我手头上的2000 sp4、XP、XP sp1、XP sp2、2003的`ntdll.dll`而言，无一例外。不过Mark Russinovich的《深入解析Windows操作系统》一书中指出，IA64的调用方式与此不同由于手头上没有相应的文件，所以在这里不进行讨论了就。

接着说。我们可以把`mov`的一句用如下的一个结构来表示：

```
#pragma pack( push, 1 )
typedef struct _tagSSDTEntry {
    BYTE  byMov;   // 0xb8
    DWORD dwIndex;
} SSDTENTRY;
#pragma pack( pop )
```

那么，我们可以对`ntdll.dll`的所有导出函数进行枚举，并筛选出“`Nt`”开头者，以`SSDTENTRY`的结构取出其开头5个字节进行比对这就是整个的枚举过程。相关的PE文件格式解析我不再解释，可参考注释。整个代码如下：

```
#define MOV        0xb8

void EnumSSDT( IN HANDLE hDriver, IN HMODULE hNtDll )
{
    DWORD dwOffset                  = (DWORD)hNtDll;
    PIMAGE_EXPORT_DIRECTORY pExpDir = NULL;
    int nNameCnt                    = 0;
    LPDWORD pNameArray              = NULL;
    int i                           = 0;

    // 到PE头部
    dwOffset += ((PIMAGE_DOS_HEADER)hNtDll)->e_lfanew + sizeof( DWORD );
    // 到第一个数据目录
    dwOffset += sizeof( IMAGE_FILE_HEADER ) + sizeof( IMAGE_OPTIONAL_HEADER )
        - IMAGE_NUMBEROF_DIRECTORY_ENTRIES * sizeof( IMAGE_DATA_DIRECTORY );
    // 到导出表位置
    dwOffset = (DWORD)hNtDll
        + ((PIMAGE_DATA_DIRECTORY)dwOffset)->VirtualAddress;
    pExpDir = (PIMAGE_EXPORT_DIRECTORY)dwOffset;

    nNameCnt = pExpDir->NumberOfNames;
    // 到函数名RVA数组
    pNameArray = (LPDWORD)( (DWORD)hNtDll + pExpDir->AddressOfNames );

    // 初始化系统模块链表
    PSYSMODULELIST pList = CreateModuleList( hNtDll );

    // 循环查找函数名
    for ( i = 0; i < nNameCnt; ++i )
    {
        PCSTR pszName = (PCSTR)( pNameArray[i] + (DWORD)hNtDll );
        if ( 'N' == pszName[0] && 't' == pszName[1] )
        {
            // 找到了函数，则定位至查找表
            LPWORD pOrdNameArray = (LPWORD)( (DWORD)hNtDll + pExpDir->AddressOfNameOrdinals );
            // 定位至总表
            LPDWORD pFuncArray   = (LPDWORD)( (DWORD)hNtDll + pExpDir->AddressOfFunctions );
            LPCVOID pFunc        = (LPCVOID)( (DWORD)hNtDll + pFuncArray[pOrdNameArray[i]] );

            // 解析函数，获取服务名
            SSDTENTRY entry;
            CopyMemory( &entry, pFunc, sizeof( SSDTENTRY ) );
            if ( MOV == entry.byMov )
            {
                ULONG ulAddr = 0;
                GetProc( hDriver, entry.dwIndex, &ulAddr );

                CHAR strModule[MAX_PATH] = "[Unknown Module]";
                FindModuleByAddr( ulAddr, pList, strModule, MAX_PATH );
                printf( "0x%04X\t%s\t0x%08X\t%s\r\n", entry.dwIndex,
                    strModule, ulAddr, pszName );
            }
        }
    }

    DestroyModuleList( pList );
}
```

下图是示例程序`SSDTDump`在XP sp2上的部分运行截图，显示了SSDT的基地址、服务个数，以及各个表项所对应的服务号、所在模块、地址和服务名。

![](https://www.kanxue.com/pediy09-645.files/ssdtdump3.gif)

## 结语

ring3与ring0，城里与城外之间为一道叹息之墙所间隔，SSDT则是越过此墙的一道必经之门。因此，很多杀毒软件也势必会围绕着它大做文章。无论是System Safety Monitor的系统监控，还是卡巴斯基的主动防御，都是挂钩了SSDT。这样，病毒尚在ring3内发作之时，便被扼杀于摇篮之内。

内核最高权限，本就是兵家必争之地，魔高一尺道高一丈的争夺于此亦已变成颇为稀松平常之事。可以说和这些争夺比起来，SSDT的相关技术简直不值一提。但最初发作的病毒体总是从ring3开始的换句话说，任你未来会成长为何等的武林高手，我都可以在你学走路的时候杀掉你知晓了SSDT的这点优势，所有的病毒咂吧咂吧也就都没味儿了。所以说么，杀毒莫如防毒。

就此打住罢，貌似扯远大发了。
