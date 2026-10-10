

# DLL 劫持 USER32

> 原文：[`www.pediy.com/kssd/pediy10/80477.html`](https://www.pediy.com/kssd/pediy10/80477.html)

你首先要了解ＰＥ，看了神奇的马甲 DLL 后,发现 XP 下可以劫持 USER32.dll，xx.exe.local（为０字节就行）,xx.exe.mainfest  
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>  
<!-- Copyright ? 1981-2001 Microsoft Corporation -->  
<assembly xmlns="urn:schemas-microsoft-com:asm.v1" manifestVersion="1.0">  
    <noInheritable/>  
    <assemblyIdentity   
        type="win32"   
        name="DllHijackerTest"   
        version="8.0.50608.0"   
        processorArchitecture="amd64"   
        publicKeyToken="1fc8b3b9a1e18e3b"  
    />  
    <file name="user32.dll"/>//关键一行  
    <file name="msvcp80.dll"/>  
    <file name="msvcm80.dll"/>  
</assembly>  
\-------------------------------------------------------------------------------------------  
hijackuser32 工程为导出所有 user32。dll  
流程：  
1 必须导出所有函数，因为 realUser32。dll 的 importTable 里的ＤＬＬ又导入ＵＳＥＲ３２.dll  
的函数.  
2,用 CreateSection 方法加载真实 DLL.  
3,在 DLLMAIN 里修复 importTable,记录真实ＥＸＰＯＲＴ函数地址．  
４，调用 realUser32（ＳＥＣＴＩＯＮ）．ＤＬＬＥＮＴＲＹＰＯＩＮＴ  
  
效果：  
在感染进程里ＬＤＲ链表没这 realUser32  
ＷＤＢＧ  
0:001> lm  
start    end        module name  
00400000 0042c000   DllHijackerTest   (deferred)               
00a10000 00a3a000   safemon    (deferred)               
10000000 1004e000   USER32     (deferred)  ／／VIRUS             
5d170000 5d20a000   comctl32\_5d170000   (deferred)               
62c20000 62c29000   LPK        (deferred)               
73640000 7366e000   msctfime   (deferred)               
73fa0000 7400b000   USP10      (deferred)               
74680000 746cb000   MSCTF      (deferred)               
76300000 7631d000   IMM32      (deferred)               
76990000 76acd000   ole32      (deferred)               
76bc0000 76bcb000   PSAPI      (deferred)               
770f0000 7717b000   OLEAUT32   (deferred)               
77180000 77283000   COMCTL32   (deferred)               
77be0000 77c38000   msvcrt     (deferred)               
77d10000 77d9f000   USER32\_77d10000   (deferred)   ／／![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")             
77da0000 77e49000   ADVAPI32   (deferred)               
77e50000 77ee2000   RPCRT4     (deferred)               
77ef0000 77f38000   GDI32      (deferred)               
77f40000 77fb6000   SHLWAPI    (deferred)               
77fc0000 77fd1000   Secur32    (deferred)               
7c800000 7c91d000   kernel32   (deferred)               
7c920000 7c9b4000   ntdll      (pdb symbols)          f:\\dbgsym\\ntdll.pdb\\36515FB5D04345E491F672FA2E2878C02\\ntdll.pdb  
7d590000 7dd83000   SHELL32    (deferred)          
       
PVOID MapDllImageBase( NT::UNICODE\_STRING\* fname )  
{  
  HANDLE hdll = NULL;  
    HANDLE hSec = NULL;  
    PVOID  dllbase = (PVOID)0;  
    
  NT::OBJECT\_ATTRIBUTES ob = {0};  
  NT::LARGE\_INTEGER off = {0};  
  ULONG viewSize = 0;  
  DWORD relen;  
  ob.Attributes = OBJ\_CASE\_INSENSITIVE;  
    ob.Length = sizeof(ob);  
  ob.ObjectName = fname;  
  NT::IO\_STATUS\_BLOCK iosb;  
  //  NT::InitializeObjectAttributes( &ob, fname, OBJ\_CASE\_INSENSITIVE, 0, 0 );  
  //  hdll = CreateFile( fname, GENERIC\_READ, FILE\_SHARE\_READ, NULL, OPEN\_EXISTING, FILE\_ATTRIBUTE\_NORMAL|FILE\_SYNCHRONOUS\_IO\_NONALERT, NULL );  
  NT::ZwOpenFile(&hdll,FILE\_READ\_DATA|FILE\_EXECUTE|SYNCHRONIZE, &ob, &iosb,  
    FILE\_SHARE\_READ, FILE\_SYNCHRONOUS\_IO\_NONALERT);  
    ob.ObjectName = NULL;  
  NT::ZwCreateSection( &hSec, SECTION\_ALL\_ACCESS, &ob, 0, PAGE\_EXECUTE, SEC\_IMAGE , hdll );  
  NT::ZwQuerySection( hSec, NT::SectionBasicInformation, &SecBaseInfo, sizeof( SecBaseInfo ), &relen );  
  NT::ZwQuerySection( hSec, NT::SectionImageInformation, &SecImageInfo, sizeof( SecImageInfo ), &relen );  
    viewSize = SecBaseInfo.Size.QuadPart;  
  NT::ZwMapViewOfSection( hSec, GetCurrentProcess(), &dllbase, 0, viewSize,  &off, &viewSize, NT::ViewShare, MEM\_TOP\_DOWN, PAGE\_EXECUTE );  
  //NT::ZwUnmapViewOfSection( GetCurrentProcess(), dllbase );  
    
  //CloseHandle( hSec );  
  CloseHandle( hdll );  
  return dllbase;  
    
};  
  
dllmain()  
{  
               h =  CreateSection  
               记录真实ＥＸＰＯＲＴ的相应地址----------A  
  FixImport( h );---------------------------B  
                (A and B 不能倒位置,呵呵,自己调试就清楚了)  
  \_\_asm push 0  
  \_\_asm push 1  
  \_\_asm push h  
  \_\_asm mov eax,entry  
  \_\_asm call eax  
}；  
我ＨＯＯＫ　并操作了　ＭＥＳＳＡＧＥＢＯＸ，![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")   
附件：  
DllHijackerTest  
hijackuser32  
ZyDllToC（自动生成部分代码）

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || hijackuser32.rar  
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || ZyDllToC.rar  
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || DllHijackerTest.rar  
 |

帮你消灭 0 回复  
  
  WCHAR image\[256\]= L"\\\\DosDevices\\\\c:\\\\windows\\\\system32\\\\user32.dll";  
尽量别这样写 用\\\\SystemRoot\\\\比较好  
  
偶发现您比较喜欢用 undocumet nt 里面的那种表示方式？  
NT::ZwQuerySection  
  
用 CreateSection 方法加载真实 DLL  
如果只加载一个 dll，其实还可以更简单 1 点  
自己分配一内存 展开 dll 的镜像   
输入表 重定位处理完毕就好了。  
可以参考偶那破文章。。  
  
在 r3 下面做事情就不需要像 r0 那么费神了  
  
manifest 偶不熟悉。。。膜拜大牛