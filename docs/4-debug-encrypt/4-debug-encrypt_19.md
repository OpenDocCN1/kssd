

# 【作品提交】Hash 计算器

> 原文：[`www.pediy.com/kssd/pediy10/68203.html`](https://www.pediy.com/kssd/pediy10/68203.html)

可能有些晚了。。。第一届软件设计大赛。。。  
  
这是一个可以计算多种 hash 的软件：CRC16/32，CRCIIT16，MD2/MD4/MD5/SHA0/SHA1  
  
可以计算某个文件，或者某个目录下的所有文件，文件大小可以超过 4GB  
  
也可以计算某个进程的某段虚拟空间，只要是可读的  
  
直接运行该程序，可以看到相关的帮助说明。  
  
F:\\>hash.exe  
  
Hash Calculator  Build 1, 2, 261, 21  
Copyright (C) 2006-2008 by Bookworm  
  
Usage:  <this EXE> \[options\] <the file or folder>  
  
All parameters are NOT case-sensitive.  
  
Options:  
  
short  long             Note  
\=====  ====             ====  
/h -h  --help           Display this help  
/r -r  --recursive      Search sub-directories as well. Default = NO  
/f -f  --file           Calculate the file's (or files under the folder) hash  
/p -p  --pid            Calculate the process' hash (user space)  
/b -b  --begin          Start address of the process, used with pid, default 0x01000000  
/e -e  --end              End address of the process, used with pid, default 0x7FFF0000  
/c -c  --CRC16  
/0 -0  --CRCIIT16  
/1 -1  --CRC32  
/2 -2  --MD2  
/3 -3  --MD4  
/4 -4  --MD5  
/5 -5  --SHA0  
/6 -6  --SHA1  
  
  
Examples:  
  
  <this EXE> -f C:\\ntdetect.com -c01   Calculate CRC16/CRCIIT16/CRC32 hash of the file  
  <this EXE> -012345f6 C:\\ntdetect.com Calculate all the hashes of the file  
  <this EXE> -012345  C:\\windows       Calculate hashes on all files under the folder  
  <this EXE> -012345r C:\\windows       Save as above, and recursive to all its sub-folders  
  <this EXE> -p 1234 -b 0x01000000 -e x02000000 -4     Get MD5 hash of the memory chunk  
                                                        in process with PID=1234  
  <this EXE> -p 1234 -4be 0x02000000 x01000000         Same as above

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || hash.rar  
 |

抱歉，这是开发的工具软件中的一个，没打算开源，故无法提供源代码。大赛并未  
规定提供完整的源代码，不算违规。遗憾的是忘了提交日期。  
  
这里有其中的一个 function，供 demo 之用。很多人写程序都是不注意细节和格式的，  
交流起来就达不到共同提高的目的。  
  
  
// \*--------------------------------------------------------------------------------------------------------------  
// copy string to clipboard  
extern HRESULT WINAPI Base\_CopyMsgToClipboardW(  
                            \_\_deref\_in      LPCWSTR pwszMsg )  
{  
    HRESULT hr = S\_OK;  
    HGLOBAL hClipBuffer = INVALID\_HANDLE\_VALUE;  
    WCHAR\*  wszChar = NULL;  
    size\_t  dwSize = 0;  
  
    if( NULL == pwszMsg )  
        return E\_INVALIDARG;  
  
    if( 0 == OpenClipboard( GetForegroundWindow() ) ||  
        0 == EmptyClipboard( ) ||  
        FAILED( hr = StringCbLengthW( pwszMsg, (DWORD)-1, &dwSize ) ) )  
    {  
        hr = HRESULT\_FROM\_LAST\_ERROR();  
        goto Done;  
    }  
  
    if( NULL == (hClipBuffer = GlobalAlloc( GMEM\_DDESHARE | GMEM\_MOVEABLE, dwSize + 10 ) ) ||  
        NULL == ( wszChar = (WCHAR\*) GlobalLock( hClipBuffer ) ) )  
    {  
        hr = HRESULT\_FROM\_LAST\_ERROR();  
        goto Done;  
    }  
  
#pragma warning( disable : 6386 )                                               // VS2005: disable warning as we know it's safe: a bug in VS2005?  
    StringCchCopyW( wszChar, (dwSize + 10)/2, pwszMsg );  
#pragma warning( default : 6386 )  
    GlobalUnlock( hClipBuffer );  
  
    if(  NULL == SetClipboardData( CF\_UNICODETEXT, hClipBuffer ) ||  
        0 == CloseClipboard() )  
        hr = HRESULT\_FROM\_LAST\_ERROR();  
Done:  
    if( INVALID\_HANDLE\_VALUE != hClipBuffer &&  
        NULL != GlobalFree( hClipBuffer ) )  
        hr = HRESULT\_FROM\_LAST\_ERROR();  
  
    return hr;  
}  
  
  
// \*--------------------------------------------------------------------------------------------------------------  
extern \_\_inline HRESULT HRESULT\_FROM\_LAST\_ERROR( VOID )  
{  
    DWORD dwError = GetLastError();  
    return HRESULT\_FROM\_WIN32( dwError );  
}