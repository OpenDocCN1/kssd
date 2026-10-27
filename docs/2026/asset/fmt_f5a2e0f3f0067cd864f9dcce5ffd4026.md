
```
static void * __stdcall SearchAPI ( char *BASE, DWORD *EAT, DWORD *ENPT, WORD *EOT, DWORD num, char *name )
{
    DWORD  index         = 0;
    char  *function_name = NULL;
    void  *function_addr = NULL;

    for ( index = 0; index < num; index++ )
    {
        function_name = ENPT[index] + BASE;

        /*
         * 大小写敏感比较
         */
        if ( 0 == strcmp( function_name, name ) )
        {
            index         = EOT[index];
            function_addr = EAT[index] + BASE;
            return( function_addr );
        }
    }  /* end of for */
    return( NULL );
}  /* end of SearchAPI */

int __cdecl main ( int argc, char * argv[] )
{
    void  *PEB                   = NULL,
          *Ldr                   = NULL,
          *Flink                 = NULL,
          *kernel32_BaseAddress  = NULL,
          *kernel32_BaseDllName  = NULL,
          *ExportDirectory       = NULL,
          *PrivateLoadLibraryA   = NULL,
          *PrivateGetProcAddress = NULL;

    DWORD  NumberOfNames         = 0,
          *AddressOfFunctions    = NULL,
          *AddressOfNames        = NULL;

    WORD  *AddressOfNameOrdinals = NULL;
    LONG   e_lfanew              = 0;

    __asm
    {
        mov     eax,fs:[0x30]
        mov     PEB,eax
    }

    printf( "PEB                   = 0x%08X\n", PEB );
    Ldr                   = *( ( void ** )( ( unsigned char * )PEB + 0x0c ) );
    printf( "Ldr                   = 0x%08X\n", Ldr );
    Flink                 = *( ( void ** )( ( unsigned char * )Ldr + 0x1c ) );
    printf( "Flink                 = 0x%08X\n", Flink );
    Flink                 = *( ( void ** )Flink );
    kernel32_BaseAddress  = *( ( void ** )( ( unsigned char * )Flink + 0x08 ) );
    kernel32_BaseDllName  = *( ( void ** )( ( unsigned char * )Flink + 0x20 ) );
    printf( "kernel32_BaseAddress  = 0x%08X\n", kernel32_BaseAddress );
    wprintf( L"kernel32_BaseDllName  = %s\n", kernel32_BaseDllName );

    /*
     * 根据PE文件格式进行解析
     */
    e_lfanew              = *( ( LONG * )( ( unsigned char * )kernel32_BaseAddress + 0x3c ) );
    printf( "e_lfanew              = 0x%08X\n", e_lfanew );
    ExportDirectory       = *( ( DWORD * )( ( unsigned char * )kernel32_BaseAddress + e_lfanew + 0x78 ) ) +
                            ( unsigned char * )kernel32_BaseAddress;
    printf( "ExportDirectory       = 0x%08X\n", ExportDirectory );
    NumberOfNames         = *( ( DWORD * )( ( unsigned char * )ExportDirectory + 0x18 ) );
    printf( "NumberOfNames         = %u\n", NumberOfNames );
    AddressOfFunctions    = ( DWORD * )
                            (
                                *( ( DWORD * )( ( unsigned char * )ExportDirectory + 0x1c ) ) +
                                ( unsigned char * )kernel32_BaseAddress
                            );
    printf( "AddressOfFunctions    = 0x%08X\n", AddressOfFunctions );
    AddressOfNames        = ( DWORD * )
                            (
                                *( ( DWORD * )( ( unsigned char * )ExportDirectory + 0x20 ) ) +
                                ( unsigned char * )kernel32_BaseAddress
                            );
    printf( "AddressOfNames        = 0x%08X\n", AddressOfNames );
    AddressOfNameOrdinals = ( WORD * )
                            (
                                *( ( DWORD * )( ( unsigned char * )ExportDirectory + 0x24 ) ) +
                                ( unsigned char * )kernel32_BaseAddress
                            );
    printf( "AddressOfNameOrdinals = 0x%08X\n", AddressOfNameOrdinals );

    PrivateLoadLibraryA   = SearchAPI
                            (
                                kernel32_BaseAddress,
                                AddressOfFunctions,
                                AddressOfNames,
                                AddressOfNameOrdinals,
                                NumberOfNames,
                                "LoadLibraryA"
                            );
    printf( "PrivateLoadLibraryA   = 0x%08X\n", PrivateLoadLibraryA );
    printf( "LoadLibraryA          = 0x%08X\n", LoadLibraryA );

    PrivateGetProcAddress = SearchAPI
                            (
                                kernel32_BaseAddress,
                                AddressOfFunctions,
                                AddressOfNames,
                                AddressOfNameOrdinals,
                                NumberOfNames,
                                "GetProcAddress"
                            );
    printf( "PrivateGetProcAddress = 0x%08X\n", PrivateGetProcAddress );
    printf( "GetProcAddress        = 0x%08X\n", GetProcAddress );

    return( EXIT_SUCCESS );
}  /* end of main */

#if 0
/*
* 按初始化顺序前向遍历链表，第二个节点对应kernel32.dll。
*/
#include <stdio.h>
#include <stdlib.h>
#pragma comment( linker, "/INCREMENTAL:NO"    )
#pragma comment( linker, "/subsystem:console" )

int __cdecl main ( int argc, char * argv[] )
{
    void *PEB         = NULL,
         *Ldr         = NULL,
         *Flink       = NULL,
         *p           = NULL,
         *BaseAddress = NULL,
         *BaseDllName = NULL;

    __asm
    {
        mov     eax,fs:[0x30]
        mov     PEB,eax
    }

    printf( "PEB   = 0x%08X\n", PEB );
    Ldr   = *( ( void ** )( ( unsigned char * )PEB + 0x0c ) );
    printf( "Ldr   = 0x%08X\n", Ldr );
    Flink = *( ( void ** )( ( unsigned char * )Ldr + 0x1c ) );
    printf( "Flink = 0x%08X\n", Flink );

    p     = Flink;
    do
    {
        BaseAddress = *( ( void ** )( ( unsigned char * )p + 0x08 ) );
        BaseDllName = *( ( void ** )( ( unsigned char * )p + 0x20 ) );
        printf( "p     = 0x%08X 0x%08X ", p, BaseAddress );
        wprintf( L"%s\n", BaseDllName );
        p = *( ( void ** )p );
    }
    while ( Flink != p );
    return( EXIT_SUCCESS );
}  /* end of main */
#endif
```

--------------------------------------------------------------------------

执行效果如下，对比`PrivateLoadLibraryA`与`LoadLibraryA`的值，应该相等。

```
> GetAddr

PEB                   = 0x7FFDF000
Ldr                   = 0x00241E90
Flink                 = 0x00241F28
kernel32_BaseAddress  = 0x77E60000
kernel32_BaseDllName  = kernel32.dll
e_lfanew              = 0x000000F8
ExportDirectory       = 0x77ECD040
NumberOfNames         = 942
AddressOfFunctions    = 0x77ECD068
AddressOfNames        = 0x77ECDF20
AddressOfNameOrdinals = 0x77ECEDD8
PrivateLoadLibraryA   = 0x77E7D961
LoadLibraryA          = 0x77E7D961
PrivateGetProcAddress = 0x77E7B332
GetProcAddress        = 0x77E7B332
```

这里演示的技巧不只用于exploit、shellcode、virus，还大量用于特殊Driver编程中。根据PE文件格式解析内存中的模块映像是很基础的知识，我今天才接触了一下，惭愧。

`SearchAPI()`用到了被搜索函数名，直接在shellcode中保存完整函数名会占用不少空间。可以考虑在shellcode中保存函数名的某种哈希值，同时`SearchAPI()`比较哈希值而非函数名。任何一种哈希算法都会丢失部分原有信息，以致不同函数名产生的哈希值相同，所谓"碰撞"。virus编程中为此常常使用标准CRC32算法([8])，参29A-4.227。但是LSD认为CRC32的汇编算法太长了([15])，他们用了一个相当简单的算法:

```
while ( *c )
{
    h = ( ( h << 5 ) | ( h >> 27 ) ) + *c++;
}
```

不超过10行汇编代码。据LSD的报告称，对超过5000个不同的dll测试，覆盖50000个不同的函数名，该哈希算法未产生一次碰撞。若真是如此，对于编写shellcode来讲，完全足够了。

下面是完整的C语言演示程序，真正汇编化后应提前计算函数名的哈希值。

--------------------------------------------------------------------------

```
/*
 * -----------------------------------------------------------------------
 * Compile : For x86/EWindows XP SP1 & VC 7
 *         : cl GetAddr_0.c /nologo /Os /G6 /W3 /D "WIN32" /D "NDEBUG" /D "_CONSOLE" /link /RELEASE
 *         :
 * Create  : 2003-08-14 17:24
 * Modify  :
 * -----------------------------------------------------------------------
*/

#include <stdio.h>
#include <stdlib.h>
#include <windows.h>

#pragma comment( linker, "/INCREMENTAL:NO"    )
#pragma comment( linker, "/subsystem:console" )

static DWORD __stdcall GetHash ( char *c )
{
    DWORD h = 0;
    while ( *c )
    {
```
