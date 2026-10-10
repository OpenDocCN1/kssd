

# PeCompact 完美脱壳笔记

> 原文：[`www.pediy.com/kssd/pediy10/103101.html`](https://www.pediy.com/kssd/pediy10/103101.html)

前段时间学习脱 PeCompact 壳，做了一些笔记。  
  
希望对初学者有所帮助。![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")  
  
PECompact 完美脱壳  
同 Aspack 一样，PECompact 也是可以被完美脱壳的。原理是：外壳完整地保留了输入表，外壳加载时没有对 IAT 加密；外壳解压数据时，完整的输入表会在内存中出现，然后外壳用显示加载 DLL 的方式获得各个函数的地址，并将该地址填充到 IAT 中。脱壳时，可以在所有的区块都解压还原后抓取内存镜像，此时外壳还没有来得及破坏原始的输入表。然后用 PE 编辑器修复抓取的镜像文件（主要是修改程序入口点，输入表）。  
举例说明。  
查壳：  
PEID    PECompact 2.x -> Jeremy Collake  
ExeInfo  PEcompact ver.2.78a ~ 3.00  
OD 设置为不忽略所有异常，载入程序。  
  
00401000 > $  B8 683B4700   mov     eax, 00473B68  
00401005   .  50            push    eax  
00401006   .  64:FF35 00000>push    dword ptr fs:\[0\]  
0040100D   .  64:8925 00000>mov     dword ptr fs:\[0\], esp  
00401014   .  33C0          xor     eax, eax  
00401016   .  8908          mov     dword ptr \[eax\], ecx  
00401018   .  50            push    eax  
  
  
  
停在外壳入口点 00401000 处。Ctrl+G 来到 00473B68 处，F2 下断。F9 运行。  
  
00401016   .  8908          mov     dword ptr \[eax\], ecx  
00401018   .  50            push    eax  
00401019   .  45            inc     ebp  
0040101A   .  43            inc     ebx  
  
  
  
在 00401016 处有写入异常。Shift+F9 运行，就会停在我们刚才下的断点处。F2 取消断点。然后 F8 单步执行。  
  
00473B68    B8 ED280B00     mov     eax, 0B28ED  
00473B6D    8D88 9E123C00   lea     ecx, dword ptr \[eax+3C129E\]  
00473B73    8941 01         mov     dword ptr \[ecx+1\], eax  
00473B76    8B5424 04       mov     edx, dword ptr \[esp+4\]  
00473B7A    8B52 0C         mov     edx, dword ptr \[edx+C\]  
00473B7D    C602 E9         mov     byte ptr \[edx\], 0E9  
00473B80    83C2 05         add     edx, 5  
00473B83    2BCA            sub     ecx, edx  
00473B85    894A FC         mov     dword ptr \[edx-4\], ecx  
00473B88    33C0            xor     eax, eax  
00473B8A    C3              retn  
  
   
  
运行到 00473B88 处，在堆栈中找到异常处理回调函数的第一个参数，在上面点右键》Follow in Dump，  
  
   
  
然后，查看数据窗口。  
  
   
第 4 个 DWORD 就是异常发生的地址。在 00401016 处下断，F9 运行，就会断在 00401016 处，F2 取消断点。F8 单步执行。  
直到遇到指令 call edi，F7 跟进去（执行外壳第二段）。  
  
00473BF2    52              push    edx  
00473BF3    8BF0            mov     esi, eax  
00473BF5    8B46 FC         mov     eax, dword ptr \[esi-4\]  
00473BF8    83C0 04         add     eax, 4  
00473BFB    2BF0            sub     esi, eax  
00473BFD    8956 08         mov     dword ptr \[esi+8\], edx  
00473C00    8B4B 0C         mov     ecx, dword ptr \[ebx+C\]  
00473C03    894E 14         mov     dword ptr \[esi+14\], ecx  
00473C06    FFD7            call    edi  //这里 F7 跟进  
00473C08    8985 3F133C00   mov     dword ptr \[ebp+3C133F\], eax  
00473C0E    8BF0            mov     esi, eax  
00473C10    8B4B 14         mov     ecx, dword ptr \[ebx+14\]  
00473C13    5A              pop     edx  
00473C14    EB 0C           jmp     short 00473C22  
  
   
  
F8 单步执行。  
  
003C1164    8985 271F3C00   mov     dword ptr \[ebp+3C1F27\], eax  
003C116A    56              push    esi  
003C116B    E8 F6030000     call    003C1566                         ; 解压各区段  
003C1170    8D8D BD1D3C00   lea     ecx, dword ptr \[ebp+3C1DBD\]  
003C1176    85C0            test    eax, eax  
003C1178    0F85 94000000   jnz     003C1212  
003C117E    56              push    esi  
003C117F    E8 40030000     call    003C14C4                         ; 修复指令 CALL 的地址  
003C1184    56              push    esi  
003C1185    E8 55020000     call    003C13DF  
003C118A    90              nop  
  
   
  
一直 F8 单步执行。  
003C1198    8B4E 34         mov     ecx, dword ptr \[esi+34\]          ; ECX＝IID 的 RVA  
003C119B    85C9            test    ecx, ecx  
003C119D    0F84 89000000   je      003C122C                         ; 双击 ZF 使其为 1,跳过外壳对 IAT 的填充  
003C11A3    034E 08         add     ecx, dword ptr \[esi+8\]  
003C11A6    51              push    ecx  
003C11A7    56              push    esi  
003C11A8    E8 47060000     call    003C17F4  
003C11AD    85C0            test    eax, eax  
003C11AF    74 7B           je      short 003C122C  
003C11B1    8B95 AA1A3C00   mov     edx, dword ptr \[ebp+3C1AAA\]  
003C11B7    8B8D AE1A3C00   mov     ecx, dword ptr \[ebp+3C1AAE\]  
  
  
   
  
（重要说明：跳过了外壳对 IAT 的填充，自然也就跳过了对 IAT 的加密。因此此种脱壳法对加密了 IAT 的依然有效）  
F8 单步执行到指令 ret。返回到外壳第一段。  
 00473C06    FFD7            call    edi                              ; 这里要 F7 跟进去  
00473C08    8985 3F133C00   mov     dword ptr \[ebp+3C133F\], eax      ; 外壳第二段返回到这里，此时 EAX 为 OEP  
00473C0E    8BF0            mov     esi, eax  
00473C10    8B4B 14         mov     ecx, dword ptr \[ebx+14\]  
00473C13    5A              pop     edx  
00473C14    EB 0C           jmp     short 00473C22  
00473C16    03CA            add     ecx, edx  
00473C18    68 00800000     push    8000  
00473C1D    6A 00           push    0  
00473C1F    57              push    edi  
00473C20    FF11            call    dword ptr \[ecx\]  
00473C22    8BC6            mov     eax, esi  
00473C24    5A              pop     edx  
00473C25    5E              pop     esi  
00473C26    5F              pop     edi  
00473C27    59              pop     ecx  
00473C28    5B              pop     ebx  
00473C29    5D              pop     ebp  
00473C2A    FFE0            jmp     eax  
  
  
  
指令 jmp eax，跳转到 OEP。此时，用 LoadPE 抓取内存镜像。  
再用 LoadPE 的 PE 编辑功能修改几个参数将输入表修改为 60000，入口点修改为 271B0。  
最后使用我编写的修复工具修复输入表就 OK 了。  
By asdfslw   \[www.pediy.com\]  
  
  
  
附件里面有 UnpackMe 和脱壳笔记，下面给出脱壳后修复程序的 C++代码（VC 6 下编译通过）  
  

代码:

```
/\*

  PeCompact 完美脱壳时抓取的内存镜像文件的输入表有些问题 IID 数组中 FirstThunk 指向的区域(IAT)全部为 0。
  这个程序用来修复输入表，修复方式很简单，就是：
  将 OriginalFirstThunk 指向的内容(INT)全部拷贝到 FirstThunk 指向的区域(IAT)。

  created by asdfslw 
  2009.11.20

\*/
#include <windows.h>


// 全局变量
HANDLE hFile;
HANDLE hFileMap;
PBYTE  pbFile;
PIMAGE\_DOS\_HEADER        pDos;
PIMAGE\_NT\_HEADERS        pNt;
PIMAGE\_SECTION\_HEADER    pSec;
PIMAGE\_IMPORT\_DESCRIPTOR pIID;

// rva 转换为 offset
DWORD RVA\_2\_Offset(DWORD rva)
{
  DWORD offset;
  PIMAGE\_SECTION\_HEADER    pSec1 = pSec + 1;
  if (rva < pSec->VirtualAddress)
  {
    offset = rva;
    return offset;
  }
  for (int i=0; i<pNt->FileHeader.NumberOfSections-1; i++)
  {
    if (rva<pSec1->VirtualAddress)
    {
      --pSec1;
      offset = rva - pSec1->VirtualAddress + pSec1->PointerToRawData;
      return offset;
    }
    ++pSec1;
  }
  --pSec1;
  offset = rva - pSec1->VirtualAddress + pSec1->PointerToRawData;
  return offset;
}

// 自定义的拷贝函数
void \_\_declspec(naked) my\_copy (DWORD dst, DWORD src)
{
  \_\_asm
  {
    push ebp
    mov ebp, esp
    push esi
    push edi
    mov edi, \[ebp+8h\]  /\* dst \*/
    mov esi, \[ebp+0ch\] /\* src \*/
\_loop\_begin:
    lods dword ptr \[esi\]
    test eax, eax
    jz \_loop\_end
    stos dword ptr \[edi\]
    jmp \_loop\_begin
\_loop\_end:
    pop edi
    pop esi
    pop ebp
    retn
  }
}


int WINAPI WinMain( HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nShowCmd )
{
  char FileName\[MAX\_PATH\] = {0};
  OPENFILENAME ofn;
  ofn.lStructSize      = sizeof(OPENFILENAME);
  ofn.hwndOwner      = 0;
  ofn.lpstrFilter      = "Exe file (\*.exe)\\0\*.exe\\00";
  ofn.lpstrCustomFilter  = NULL;
  ofn.nFilterIndex    = 1;
  ofn.lpstrFile      = FileName;
  ofn.nMaxFile      = sizeof FileName;
  ofn.lpstrFileTitle    = NULL;
  ofn.nMaxFileTitle    = 0;
  ofn.lpstrInitialDir    = NULL;
  ofn.lpstrTitle      = "Pecompact 完美脱壳修复工具 by asdfslw";
  ofn.Flags        = OFN\_EXPLORER | OFN\_PATHMUSTEXIST | OFN\_FILEMUSTEXIST | OFN\_HIDEREADONLY | OFN\_ENABLESIZING;
  ofn.lpstrDefExt      = NULL;

  if(0==GetOpenFileName(&ofn))
  {
    goto \_error;
  }


  // 打开文件
  hFile = CreateFileA(FileName, GENERIC\_READ | GENERIC\_WRITE, NULL, NULL, 
    OPEN\_EXISTING, FILE\_ATTRIBUTE\_NORMAL, NULL);
  if (INVALID\_HANDLE\_VALUE==hFile)
  {
    MessageBoxA(NULL, "打开文件错误！", "错误", MB\_ICONEXCLAMATION);
    goto \_error;
  }

  // 创建文件映射
  hFileMap = CreateFileMappingA(hFile, NULL, PAGE\_READWRITE, NULL, NULL, NULL);
  if (NULL==hFileMap)
  {
    MessageBoxA(NULL, "创建文件映射错误！", "错误", MB\_ICONEXCLAMATION);
    CloseHandle(hFile);
    goto \_error;
  }

  // 将文件数据映射到进程地址空间
  pbFile = (PBYTE)MapViewOfFile(hFileMap, FILE\_MAP\_WRITE, 0, 0, 0);
  if (NULL==pbFile)
  {
    MessageBoxA(NULL, "将文件数据映射到进程地址空间错误！", "错误", MB\_ICONEXCLAMATION);
    CloseHandle(hFileMap);
    CloseHandle(hFile);
    goto \_error;
  }

  // 获取 PE 结构相关数据
  pDos = (PIMAGE\_DOS\_HEADER)     pbFile;
  pNt  = (PIMAGE\_NT\_HEADERS)     ( (PBYTE)pDos + pDos->e\_lfanew );
  pSec = (PIMAGE\_SECTION\_HEADER) ( (PBYTE)pNt + 0xF8 );
  pIID = (PIMAGE\_IMPORT\_DESCRIPTOR) (
    RVA\_2\_Offset(pNt->OptionalHeader.DataDirectory\[1\].VirtualAddress) 
    + (DWORD)pbFile );

  while (NULL!=pIID->Name)
  {
    my\_copy( 
      RVA\_2\_Offset(pIID->FirstThunk)+ (DWORD)pbFile, 
      RVA\_2\_Offset(pIID->OriginalFirstThunk) + (DWORD)pbFile );
    ++pIID;
  }
  
  MessageBoxA(NULL, "成功", "呵呵", MB\_OK);

  UnmapViewOfFile(pbFile);
  CloseHandle(hFileMap);
  CloseHandle(hFile);
  return 0;
\_error:
  ExitProcess(0);
}

```

  
本人在脱壳方面也是一个初学者。笔记中有什么不对的地方，希望各位高手指正。谢谢![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")  
  
另外，大家可以参考看雪如下文章  
[`bbs.pediy.com/showthread.php?t=10932`](http://bbs.pediy.com/showthread.php?t=10932)  
这篇文章的思路与我的相同（不过写这篇笔记之前我并没有看到这篇文章，之后才搜索到的）。但遗憾的是这篇文章里面没有对脱壳后的文件的输入表进行修复，我则找出了脱壳后的文件不能运行的问题所在输入表有问题，并编写了一个修复程序（虽然修复其实很简单）。  
  
【版权声明】本文由 asdfslw 原创于看雪软件安全论坛，转载请注明出处，并保留版权声明。  
  
  
更新              ：添加了修复程序 PeCompact\_IAT\_xiufu.rar。  
关于修复程序的说明：如果你的电脑上安装有主动防御类软件或者其他安全软件，在修复程序运行的时候可能会报警。比如说，我的电脑上安装有微点主动防御软件，修复程序会修改可执行文件，默认情况下微点对修改可执行文件的程序都会报警。此时只要点允许，并把修复程序添加到可信程序中，重新运行一次修复程序就可以了。本人保证此修复程序是没有任何危害的。 

我以前写过一个 PeCompact 的脚本，效果应该相当于静态脱壳机：  
  
http://bbs.pediy.com/showthread.php?t=83885 

msg "已经到 OEP 了。程序已 dump 后另存为 UN\_+原文件名。OEP 、输入表、重定位（若有）都已修正，区段都已重建。若要优化大小，请查看输出表及资源是否在最后那个“CanBeDel”段中，若在，请重建输出表及资源后再删除最后的那个“CanBeDel”段。"  
  
"若在，请重建输出表及资源后再删除最后的那个“CanBeDel”段。"  
怎么重建输入表及资源？![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")  
  
另外，想请教下 OD 脚本怎么编写？哪里有教程或者资料？ 

引用: *最初由 **asdfslw**发布* ![查看帖子](img/78d57e1ac1f1fc075f84e78a6a789d46.jpg)msg "已经到 OEP 了。程序已 dump 后另存为 UN\_+原文件名。OEP 、输入表、重定位（若有）都已修正，区段都已重建。若要优化大小，请查看输出表及资源是否在最后那个“CanBeDel”段中，若在，请重建输出表及资源后再删除最后的那个“CanBeDel”段。"  
  
&quot... 

不需要重建输入表，输入表已经建好了。脱出来的程序直接就可以运行。如果要优化大小的话，对于 DLL 这样的有输出表的，可能要把输出表从那个“CanBeDel”段中移到别的地方去（可以用 kanxue 兄的那个 pemove 小工具来完成），资源直接用 FixRes dump 一份，再删掉那个“CanBeDel”段，把 dump 后的资源导入到最后。  
  
脚本编写可以参考 hnhuqiong 的教程，这个帖子上有列表：  
  
http://bbs.pediy.com/showpost.php?p=514664&postcount=2  
  
另外就是看别人的脚本，然后在 OD 中单步跑一遍，基本上就清楚了。只要会用 OD，再了解一下脚本命令，应该就可以写出脚本。