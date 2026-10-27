
```
PEB_LDR_DATA STRUCT                 ; sizeof = 24h

    _Length                         DWORD       ?   ; original name Length

    Initialized                     BYTE        ?   ; 04h

                                    db  3 dup(?)    ; padding

    SsHandle                        PVOID       ?   ; 08h

    InLoadOrderModuleList           LIST_ENTRY  <>  ; 0Ch

    InMemoryOrderModuleList         LIST_ENTRY  <>  ; 14h

    InInitializationOrderModuleList LIST_ENTRY  <>  ; 1Ch

PEB_LDR_DATA ENDS

PPEB_LDR_DATA typedef PTR PEB_LDR_DATA



LIST_ENTRY STRUCT

    Flink DWORD ?

    Blink DWORD ?

LIST_ENTRY ENDS



LDR_DATA_TABLE_ENTRY_U0 union ; (sizeof=0X8, standard type)

    HashLinks                       LIST_ENTRY     <>

    SectionPointer                  DWORD          ?       ; offset (FFFFFFFF)

LDR_DATA_TABLE_ENTRY_U0 ends



LDR_DATA_TABLE_ENTRY_U1 union ; (sizeof=0X4, standard type)

    TimeDateStamp                   DWORD          ?

    LoadedImports                   DWORD          ?       ; offset (FFFFFFFF)

LDR_DATA_TABLE_ENTRY_U1 ends



LDR_DATA_TABLE_ENTRY struc ; (sizeof=0X54, standard type)

    InLoadOrderModuleList           LIST_ENTRY     <>

    InMemoryOrderModuleList         LIST_ENTRY     <>

    InInitializationOrderModuleList LIST_ENTRY     <>

    DllBase                         DWORD          ?       ; offset (FFFFFFFF)

    EntryPoint                      DWORD          ?       ; offset (FFFFFFFF)

    SizeOfImage                     DWORD          ?

    FullDllName                     UNICODE_STRING <>

    BaseDllName                     UNICODE_STRING <>

    Flags                           DWORD          ?

    LoadCount                       WORD           ?

    TlsIndex                        WORD           ?

    U0                              LDR_DATA_TABLE_ENTRY_U0 <>

    CheckSum                        DWORD          ?

    U1                              LDR_DATA_TABLE_ENTRY_U1 <>

    EntryPointActivationContext     DWORD          ?       ; offset (FFFFFFFF)

    PatchInformation                DWORD          ?       ; offset (FFFFFFFF)

LDR_DATA_TABLE_ENTRY ends
```

`LDR_DATA_TABLE_ENTRY`结构的前三个元素都是链表结构，定义成`LIST_ENTRY`结构，通过`LIST_ENTRY.Flink`（向前链界）或`LIST_ENTRY.Blink`（向后链接），和下一个`LDR_DATA_TABLE_ENTRY`链接，因此通过这个结构，可以遍列所有加载的模块，我们就是根据这个特性来得到所需模块的基地址。

`PEB.Ldr`指向`PEB_LDR_DATA`，`PEB_LDR_DATA.InLoadOrderModuleList.Flink`就指向我们关心的`LDR_DATA_TABLE_ENTRY`表。

各位还记得上一篇中那个“象鼻子吸盘”的代码吗，这段代码最后返回的是某个函数的序号，`AddrOfNameOrdinalsBase`序号表单位是`WORD`，根据这个序号，在`AddressOfFunctions`相应的位置上就是函数的调用地址了，就这么简单，只要理解了输出表的结构中几个元素的含义，就非常容易打造我们自己的函数：
