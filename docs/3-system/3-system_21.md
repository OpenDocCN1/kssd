

# PE 教程 6: Import Table（引入表）

> 原文：[`www.pediy.com/kssd/tutorial/chap8-1-6.htm`](https://www.pediy.com/kssd/tutorial/chap8-1-6.htm)

 

# PE 教程 6: Import Table（引入表）

本课我们将学习引入表。先警告一下，对于不熟悉引入表的读者来说，这是一堂又长又难的课，所以需要多读几遍，最好再打开调试器来好好分析相关结构。各位，努力啊！

下载**范例**。

### 理论:

首先，您得了解什么是引入函数。一个引入函数是被某模块调用的但又不在调用者模块中的函数，因而命名为"import（引入）"。引入函数实际位于一个或者更多的 DLL 里。调用者模块里只保留一些函数信息，包括函数名及其驻留的 DLL 名。现在，我们怎样才能找到 PE 文件中保存的信息呢? 转到**data directory** 寻求答案吧。再回顾一把，下面就是 PE header:

> IMAGE\_NT\_HEADERS STRUCT  
>    Signature dd ?  
>    FileHeader IMAGE\_FILE\_HEADER <>  
>    OptionalHeader IMAGE\_OPTIONAL\_HEADER <>  
> IMAGE\_NT\_HEADERS ENDS

optional header 最后一个成员就是 data directory（数据目录）:

IMAGE\_OPTIONAL\_HEADER32 STRUCT  
   ....   
   LoaderFlags dd ?   
   NumberOfRvaAndSizes dd ?   
   DataDirectory IMAGE\_DATA\_DIRECTORY 16 dup(<>)   
IMAGE\_OPTIONAL\_HEADER32 ENDS 

data directory 是一个 **IMAGE\_DATA\_DIRECTORY** 结构数组，共有 16 个成员。如果您还记得节表可以看作是 PE 文件各节的根目录的话，也可以认为 data directory 是存储在这些节里的逻辑元素的根目录。明确点，data directory 包含了 PE 文件中各重要数据结构的位置和尺寸信息。 每个成员包含了一个重要数据结构的信息。

| Member  || Info inside |
| --- | --- |
| 0  || Export symbols |
| 1  || Import symbols |
| 2  || Resources |
| 3  || Exception |
| 4  || Security |
| 5  || Base relocation |
| 6  || Debug |
| 7  || Copyright string |
| 8  || Unknown |
| 9  || Thread local storage (TLS) |
| 10  || Load configuration |
| 11  || Bound Import |
| 12  || Import Address Table |
| 13  || Delay Import |
| 14  || COM descriptor |

上面那些金色显示的是我熟悉的。了解 data directory 包含域后，我们可以仔细研究它们了。data directory 的每个成员都是 **IMAGE\_DATA\_DIRECTORY** 结构类型的，其定义如下所示:

IMAGE\_DATA\_DIRECTORY STRUCT   
  VirtualAddress dd ?   
  isize dd ?   
IMAGE\_DATA\_DIRECTORY ENDS 

**VirtualAddress** 实际上是数据结构的相对虚拟地址(RVA)。比如，如果该结构是关于 import symbols 的，该域就包含指向**IMAGE\_IMPORT\_DESCRIPTOR **数组的 RVA。   
**isize **含有**VirtualAddress**所指向数据结构的字节数。

下面就是如何找寻 PE 文件中重要数据结构的一般方法:

1.  从 DOS header 定位到 PE header
2.  从 optional header 读取 data directory 的地址。
3.  **IMAGE\_DATA\_DIRECTORY **结构尺寸乘上找寻结构的索引号: 比如您要找寻 import symbols 的位置信息，必须用**IMAGE\_DATA\_DIRECTORY** 结构尺寸(8 bytes)乘上 1（import symbols 在 data directory 中的索引号）。
4.  将上面的结果加上 data directory 地址，我们就得到包含所查询数据结构信息的 **IMAGE\_DATA\_DIRECTORY** 结构项。

现在我们开始真正讨论引入表了。data directory 数组第二项的**VirtualAddress**包含引入表地址。引入表实际上是一个 **IMAGE\_IMPORT\_DESCRIPTOR** 结构数组。每个结构包含 PE 文件引入函数的一个相关 DLL 的信息。比如，如果该 PE 文件从 10 个不同的 DLL 中引入函数，那么这个数组就有 10 个成员。该数组以一个全 0 的成员结尾。下面详细研究结构组成:

IMAGE\_IMPORT\_DESCRIPTOR STRUCT   
  union   
    Characteristics dd ?   
    OriginalFirstThunk dd ?   
  ends   
  TimeDateStamp dd ?   
  ForwarderChain dd ?   
  Name1 dd ?   
  FirstThunk dd ?   
IMAGE\_IMPORT\_DESCRIPTOR ENDS 

结构第一项是一个 union 子结构。 事实上，这个 union 子结构只是给 **OriginalFirstThunk** 增添了个别名，您也可以称其为"Characteristics"。 该成员项含有指向一个 **IMAGE\_THUNK\_DATA** 结构数组的 RVA。  
什么是 **IMAGE\_THUNK\_DATA**? 这是一个 dword 类型的集合。通常我们将其解释为指向一个 **IMAGE\_IMPORT\_BY\_NAME** 结构的指针。注意 **IMAGE\_THUNK\_DATA** 包含了指向一个 **IMAGE\_IMPORT\_BY\_NAME** 结构的指针: 而不是结构本身。  
请看这里: 现有几个 **IMAGE\_IMPORT\_BY\_NAME** 结构，我们收集起这些结构的 RVA (**IMAGE\_THUNK\_DATAs**)组成一个数组，并以 0 结尾，然后再将数组的 RVA 放入 **OriginalFirstThunk**。  
此 **IMAGE\_IMPORT\_BY\_NAME** 结构存有一个引入函数的相关信息。再来研究 **IMAGE\_IMPORT\_BY\_NAME** 结构到底是什么样子的呢:

IMAGE\_IMPORT\_BY\_NAME STRUCT   
  Hint dw ?   
  Name1 db ?   
IMAGE\_IMPORT\_BY\_NAME ENDS 

**Hint **指示本函数在其所驻留 DLL 的引出表中的索引号。该域被 PE 装载器用来在 DLL 的引出表里快速查询函数。该值不是必须的，一些连接器将此值设为 0。  
**Name1** 含有引入函数的函数名。函数名是一个 ASCIIZ 字符串。注意这里虽然将 Name1 的大小定义成字节，其实它是可变尺寸域，只不过我们没有更好方法来表示结构中的可变尺寸域。The structure is provided so that you can refer to the data structure with descriptive names.  

**TimeDateStamp** 和 **ForwarderChain** 可是高级东东: 让我们精通其他成员后再来讨论它们吧。

**Name1** 含有指向 DLL 名字的 RVA，即指向 DLL 名字的指针，也是一个 ASCIIZ 字符串。

**FirstThunk** 与 **OriginalFirstThunk** 非常相似，它也包含指向一个 **IMAGE\_THUNK\_DATA** 结构数组的 RVA(当然这是另外一个**IMAGE\_THUNK\_DATA** 结构数组)。   
好了，如果您还在犯糊涂，就朝这边看过来: 现在有几个 **IMAGE\_IMPORT\_BY\_NAME **结构，同时您又创建了两个结构数组，并同样寸入指向那些 **IMAGE\_IMPORT\_BY\_NAME** 结构的 RVAs，这样两个数组就包含相同数值了(可谓相当精确的复制啊)。 最后您决定将第一个数组的 RVA 赋给 **OriginalFirstThunk****，**第二个数组的 RVA 赋给 **FirstThunk**，这样一切都很清楚了。

| OriginalFirstThunk || 　 || IMAGE\_IMPORT\_BY\_NAME || 　 || FirstThunk |
| --- | --- | --- | --- | --- |
| |  || 　 || 　 || 　 || |  |
| 

| IMAGE\_THUNK\_DATA  |
| :-: |
| IMAGE\_THUNK\_DATA  |
| IMAGE\_THUNK\_DATA  |
| IMAGE\_THUNK\_DATA  |
| ...  |
| IMAGE\_THUNK\_DATA  |

 || 

| \---> |
| :-: |
| \---> |
| \---> |
| \---> |
| \---> |
| \---> |

 || 

| Function 1  |
| :-: |
| Function 2  |
| Function 3  |
| Function 4  |
| ...  |
| Function n  |

 || 

| <--- |
| :-: |
| <--- |
| <--- |
| <--- |
| <--- |
| <--- |

 || 

| IMAGE\_THUNK\_DATA  |
| :-: |
| IMAGE\_THUNK\_DATA  |
| IMAGE\_THUNK\_DATA  |
| IMAGE\_THUNK\_DATA  |
| ...  |
| IMAGE\_THUNK\_DATA  |

 |

现在您应该明白我的意思。不要被**IMAGE\_THUNK\_DATA**这个名字弄糊涂: 它仅是指向 **IMAGE\_IMPORT\_BY\_NAME **结构的 RVA。 如果将 **IMAGE\_THUNK\_DATA** 字眼想象成 RVA，就更容易明白了。**OriginalFirstThunk** 和 **FirstThunk** 所指向的这两个数组大小取决于 PE 文件从 DLL 中引入函数的数目。比如，如果 PE 文件从 kernel32.dll 中引入 10 个函数，那么**IMAGE\_IMPORT\_DESCRIPTOR** 结构的 **Name1**域包含指向字符串"kernel32.dll"的 RVA，同时每个**IMAGE\_THUNK\_DATA** 数组有 10 个元素。

下一个问题是: 为什么我们需要两个完全相同的数组? 为了回答该问题，我们需要了解当 PE 文件被装载到内存时，PE 装载器将查找**IMAGE\_THUNK\_DATA** 和 **IMAGE\_IMPORT\_BY\_NAME** 这些结构数组，以此决定引入函数的地址。然后用引入函数真实地址来替代由**FirstThunk**指向的**IMAGE\_THUNK\_DATA** 数组里的元素值。因此当 PE 文件准备执行时，上图已转换成:

| OriginalFirstThunk || 　 || IMAGE\_IMPORT\_BY\_NAME || 　 || FirstThunk |
| --- | --- | --- | --- | --- |
| |  || 　 || 　 || 　 || |  |
| 

| IMAGE\_THUNK\_DATA  |
| :-: |
| IMAGE\_THUNK\_DATA  |
| IMAGE\_THUNK\_DATA  |
| IMAGE\_THUNK\_DATA  |
| ...  |
| IMAGE\_THUNK\_DATA  |

 || 

| \---> |
| :-: |
| \---> |
| \---> |
| \---> |
| \---> |
| \---> |

 || 

| Function 1  |
| :-: |
| Function 2  |
| Function 3  |
| Function 4  |
| ...  |
| Function n  |

 || 

|     |
| :-: |
| 　 |
| 　 |
| 　 |
| 　 |
| 　 |

 || 

| Address of Function 1  |
| :-: |
| Address of Function 2  |
| Address of Function 3  |
| Address of Function 4  |
| ...  |
| Address of Function n  |

 |

由**OriginalFirstThunk** 指向的 RVA 数组始终不会改变，所以若还反过头来查找引入函数名，PE 装载器还能找寻到。  
当然再简单的事物都有其复杂的一面。有些情况下一些函数仅由序数引出，也就是说您不能用函数名来调用它们: 您只能用它们的位置来调用。此时，调用者模块中就不存在该函数的**IMAGE\_IMPORT\_BY\_NAME** 结构。不同的，对应该函数的 **IMAGE\_THUNK\_DATA** 值的低位字指示函数序数，而最高二进位 (MSB)设为 1。例如，如果一个函数只由序数引出且其序数是 1234h，那么对应该函数的 **IMAGE\_THUNK\_DATA** 值是 80001234h。Microsoft 提供了一个方便的常量来测试 dword 值的 MSB 位，就是 **IMAGE\_ORDINAL\_FLAG32**，其值为 80000000h。  
假设我们要列出某个 PE 文件的所有引入函数，可以照着下面步骤走:

1.  校验文件是否是有效的 PE。
2.  从 DOS header 定位到 PE header。
3.  获取位于 **OptionalHeader **数据目录地址。
4.  转至数据目录的第二个成员提取其**VirtualAddress**值。
5.  利用上值定位第一个 **IMAGE\_IMPORT\_DESCRIPTOR** 结构。
6.  检查 **OriginalFirstThunk**值。若不为 0，顺着 **OriginalFirstThunk** 里的 RVA 值转入那个 RVA 数组。若 **OriginalFirstThunk** 为 0，就改用**FirstThunk**值。有些连接器生成 PE 文件时会置**OriginalFirstThunk**值为 0，这应该算是个 bug。不过为了安全起见，我们还是检查 **OriginalFirstThunk**值先。
7.  对于每个数组元素，我们比对元素值是否等于**IMAGE\_ORDINAL\_FLAG32****。**如果该元素值的最高二进位为 1， 那么函数是由序数引入的，可以从该值的低字节提取序数。
8.  如果元素值的最高二进位为 0，就可将该值作为 RVA 转入 **IMAGE\_IMPORT\_BY\_NAME** 数组，跳过 **Hint** 就是函数名字了。
9.  再跳至下一个数组元素提取函数名一直到数组底部(它以 null 结尾)。现在我们已遍历完一个 DLL 的引入函数，接下去处理下一个 DLL。
10.  即跳转到下一个 **IMAGE\_IMPORT\_DESCRIPTOR** 并处理之，如此这般循环直到数组见底。(**IMAGE\_IMPORT\_DESCRIPTOR** 数组以一个全 0 域元素结尾)。

### 示例:

本例程打开一 PE 文件，将所有引入函数名读入一编辑控件，同时显示 **IMAGE\_IMPORT\_DESCRIPTOR** 结构各域值。

.386   
.model flat,stdcall   
option casemap:none   
include \\masm32\\include\\windows.inc   
include \\masm32\\include\\kernel32.inc   
include \\masm32\\include\\comdlg32.inc   
include \\masm32\\include\\user32.inc   
includelib \\masm32\\lib\\user32.lib   
includelib \\masm32\\lib\\kernel32.lib   
includelib \\masm32\\lib\\comdlg32.lib   
  
IDD\_MAINDLG equ 101   
IDC\_EDIT equ 1000   
IDM\_OPEN equ 40001   
IDM\_EXIT equ 40003   
  
DlgProc proto :DWORD,:DWORD,:DWORD,:DWORD   
ShowImportFunctions proto :DWORD   
ShowTheFunctions proto :DWORD,:DWORD   
AppendText proto :DWORD,:DWORD   
  
SEH struct   
PrevLink dd ? ; the address of the previous seh structure   
CurrentHandler dd ? ; the address of the new exception handler   
SafeOffset dd ? ; The offset where it's safe to continue execution   
PrevEsp dd ? ; the old value in esp   
PrevEbp dd ? ; The old value in ebp   
SEH ends   
  
.data   
AppName db "PE tutorial no.6",0   
ofn OPENFILENAME <>   
FilterString db "Executable Files (\*.exe, \*.dll)",0,"\*.exe;\*.dll",0   
             db "All Files",0,"\*.\*",0,0   
FileOpenError db "Cannot open the file for reading",0   
FileOpenMappingError db "Cannot open the file for memory mapping",0   
FileMappingError db "Cannot map the file into memory",0   
NotValidPE db "This file is not a valid PE",0   
CRLF db 0Dh,0Ah,0   
ImportDescriptor db 0Dh,0Ah,"================\[ IMAGE\_IMPORT\_DESCRIPTOR \]=============",0   
IDTemplate db "OriginalFirstThunk = %lX",0Dh,0Ah   
           db "TimeDateStamp = %lX",0Dh,0Ah   
           db "ForwarderChain = %lX",0Dh,0Ah   
           db "Name = %s",0Dh,0Ah   
           db "FirstThunk = %lX",0   
NameHeader db 0Dh,0Ah,"Hint Function",0Dh,0Ah   
           db "-----------------------------------------",0   
NameTemplate db "%u %s",0   
OrdinalTemplate db "%u (ord.)",0   
  
.data?   
buffer db 512 dup(?)   
hFile dd ?   
hMapping dd ?   
pMapping dd ?   
ValidPE dd ?   
  
.code   
start:   
invoke GetModuleHandle,NULL   
invoke DialogBoxParam, eax, IDD\_MAINDLG,NULL,addr DlgProc, 0   
invoke ExitProcess, 0   
  
DlgProc proc hDlg:DWORD, uMsg:DWORD, wParam:DWORD, lParam:DWORD   
.if uMsg==WM\_INITDIALOG   
  invoke SendDlgItemMessage,hDlg,IDC\_EDIT,EM\_SETLIMITTEXT,0,0   
.elseif uMsg==WM\_CLOSE   
  invoke EndDialog,hDlg,0   
.elseif uMsg==WM\_COMMAND   
  .if lParam==0   
    mov eax,wParam   
    .if ax==IDM\_OPEN   
      invoke ShowImportFunctions,hDlg   
    .else ; IDM\_EXIT   
      invoke SendMessage,hDlg,WM\_CLOSE,0,0   
    .endif   
  .endif   
.else   
  mov eax,FALSE   
  ret   
.endif   
mov eax,TRUE   
ret   
DlgProc endp   
  
SEHHandler proc uses edx pExcept:DWORD, pFrame:DWORD, pContext:DWORD, pDispatch:DWORD   
  mov edx,pFrame   
  assume edx:ptr SEH   
  mov eax,pContext   
  assume eax:ptr CONTEXT   
  push \[edx\].SafeOffset   
  pop \[eax\].regEip   
  push \[edx\].PrevEsp   
  pop \[eax\].regEsp   
  push \[edx\].PrevEbp   
  pop \[eax\].regEbp   
  mov ValidPE, FALSE   
  mov eax,ExceptionContinueExecution   
  ret   
SEHHandler endp   
  
ShowImportFunctions proc uses edi hDlg:DWORD   
  LOCAL seh:SEH   
  mov ofn.lStructSize,SIZEOF   
  ofn mov ofn.lpstrFilter, OFFSET FilterString   
  mov ofn.lpstrFile, OFFSET buffer   
  mov ofn.nMaxFile,512   
  mov ofn.Flags, OFN\_FILEMUSTEXIST or OFN\_PATHMUSTEXIST or OFN\_LONGNAMES or OFN\_EXPLORER or OFN\_HIDEREADONLY   
  invoke GetOpenFileName, ADDR ofn   
  .if eax==TRUE   
    invoke CreateFile, addr buffer, GENERIC\_READ, FILE\_SHARE\_READ, NULL, OPEN\_EXISTING, FILE\_ATTRIBUTE\_NORMAL, NULL   
    .if eax!=INVALID\_HANDLE\_VALUE   
      mov hFile, eax   
      invoke CreateFileMapping, hFile, NULL, PAGE\_READONLY,0,0,0   
      .if eax!=NULL   
        mov hMapping, eax   
        invoke MapViewOfFile,hMapping,FILE\_MAP\_READ,0,0,0   
        .if eax!=NULL   
          mov pMapping,eax   
          assume fs:nothing   
          push fs:\[0\]   
          pop seh.PrevLink   
          mov seh.CurrentHandler,offset SEHHandler   
          mov seh.SafeOffset,offset FinalExit   
          lea eax,seh   
          mov fs:\[0\], eax   
          mov seh.PrevEsp,esp   
          mov seh.PrevEbp,ebp   
          mov edi, pMapping   
          assume edi:ptr IMAGE\_DOS\_HEADER   
          .if \[edi\].e\_magic==IMAGE\_DOS\_SIGNATURE   
            add edi, \[edi\].e\_lfanew   
            assume edi:ptr IMAGE\_NT\_HEADERS   
            .if \[edi\].Signature==IMAGE\_NT\_SIGNATURE   
              mov ValidPE, TRUE   
            .else   
              mov ValidPE, FALSE   
            .endif   
          .else   
            mov ValidPE,FALSE   
          .endif   
FinalExit:   
          push seh.PrevLink   
          pop fs:\[0\]   
          .if ValidPE==TRUE   
            invoke ShowTheFunctions, hDlg, edi   
          .else   
            invoke MessageBox,0, addr NotValidPE, addr AppName, MB\_OK+MB\_ICONERROR   
          .endif   
          invoke UnmapViewOfFile, pMapping   
      .else   
          invoke MessageBox, 0, addr FileMappingError, addr AppName, MB\_OK+MB\_ICONERROR   
      .endif   
      invoke CloseHandle,hMapping   
    .else   
      invoke MessageBox, 0, addr FileOpenMappingError, addr AppName, MB\_OK+MB\_ICONERROR   
    .endif   
    invoke CloseHandle, hFile   
   .else   
   invoke MessageBox, 0, addr FileOpenError, addr AppName, MB\_OK+MB\_ICONERROR   
   .endif   
 .endif   
 ret   
ShowImportFunctions endp   
  
AppendText proc hDlg:DWORD,pText:DWORD   
   invoke SendDlgItemMessage,hDlg,IDC\_EDIT,EM\_REPLACESEL,0,pText   
   invoke SendDlgItemMessage,hDlg,IDC\_EDIT,EM\_REPLACESEL,0,addr CRLF   
   invoke SendDlgItemMessage,hDlg,IDC\_EDIT,EM\_SETSEL,-1,0   
   ret   
AppendText endp   
  
RVAToOffset PROC uses edi esi edx ecx pFileMap:DWORD,RVA:DWORD   
   mov esi,pFileMap   
   assume esi:ptr IMAGE\_DOS\_HEADER   
   add esi,\[esi\].e\_lfanew   
   assume esi:ptr IMAGE\_NT\_HEADERS   
   mov edi,RVA ; edi == RVA   
   mov edx,esi   
   add edx,sizeof IMAGE\_NT\_HEADERS   
   mov cx,\[esi\].FileHeader.NumberOfSections   
   movzx ecx,cx   
   assume edx:ptr IMAGE\_SECTION\_HEADER   
   .while ecx>0 ; check all sections   
     .if edi>=\[edx\].VirtualAddress   
       mov eax,\[edx\].VirtualAddress   
       add eax,\[edx\].SizeOfRawData   
       .if edi<eax ; The address is in this section   
         mov eax,\[edx\].VirtualAddress   
         sub edi,eax  
         mov eax,\[edx\].PointerToRawData   
         add eax,edi ; eax == file offset   
         ret   
       .endif   
     .endif   
     add edx,sizeof IMAGE\_SECTION\_HEADER   
     dec ecx   
   .endw   
   assume edx:nothing   
   assume esi:nothing   
   mov eax,edi   
   ret   
RVAToOffset endp   
  
ShowTheFunctions proc uses esi ecx ebx hDlg:DWORD, pNTHdr:DWORD   
   LOCAL temp\[512\]:BYTE   
   invoke SetDlgItemText,hDlg,IDC\_EDIT,0   
   invoke AppendText,hDlg,addr buffer   
   mov edi,pNTHdr   
   assume edi:ptr IMAGE\_NT\_HEADERS   
   mov edi, \[edi\].OptionalHeader.DataDirectory\[sizeof IMAGE\_DATA\_DIRECTORY\].VirtualAddress   
   invoke RVAToOffset,pMapping,edi   
   mov edi,eax   
   add edi,pMapping   
   assume edi:ptr IMAGE\_IMPORT\_DESCRIPTOR   
   .while !(\[edi\].OriginalFirstThunk==0 && \[edi\].TimeDateStamp==0 && \[edi\].ForwarderChain==0 && \[edi\].Name1==0 && \[edi\].FirstThunk==0)   
     invoke AppendText,hDlg,addr ImportDescriptor   
     invoke RVAToOffset,pMapping, \[edi\].Name1   
     mov edx,eax   
     add edx,pMapping   
     invoke wsprintf, addr temp, addr IDTemplate, \[edi\].OriginalFirstThunk,\[edi\].TimeDateStamp,\[edi\].ForwarderChain,edx,\[edi\].FirstThunk      invoke AppendText,hDlg,addr temp   
     .if \[edi\].OriginalFirstThunk==0   
        mov esi,\[edi\].FirstThunk   
     .else   
        mov esi,\[edi\].OriginalFirstThunk   
     .endif   
     invoke RVAToOffset,pMapping,esi   
     add eax,pMapping   
     mov esi,eax   
     invoke AppendText,hDlg,addr NameHeader   
     .while dword ptr \[esi\]!=0   
       test dword ptr \[esi\],IMAGE\_ORDINAL\_FLAG32   
       jnz ImportByOrdinal   
       invoke RVAToOffset,pMapping,dword ptr \[esi\]   
       mov edx,eax   
       add edx,pMapping   
       assume edx:ptr IMAGE\_IMPORT\_BY\_NAME   
       mov cx, \[edx\].Hint   
       movzx ecx,cx   
       invoke wsprintf,addr temp,addr NameTemplate,ecx,addr \[edx\].Name1   
       jmp ShowTheText   
ImportByOrdinal:   
       mov edx,dword ptr \[esi\]   
       and edx,0FFFFh   
       invoke wsprintf,addr temp,addr OrdinalTemplate,edx   
ShowTheText:   
       invoke AppendText,hDlg,addr temp   
       add esi,4   
    .endw   
    add edi,sizeof IMAGE\_IMPORT\_DESCRIPTOR   
  .endw   
  ret   
ShowTheFunctions endp   
end start 

### 分析:

本例中，用户点击打开菜单显示文件打开对话框，检验文件的 PE 有效性后调用 **ShowTheFunctions**。

ShowTheFunctions proc uses esi ecx ebx hDlg:DWORD, pNTHdr:DWORD   
   LOCAL temp\[512\]:BYTE 

保留 512 字节堆栈空间用于字符串操作。

   invoke SetDlgItemText,hDlg,IDC\_EDIT,0 

清除编辑控件内容。

   invoke AppendText,hDlg,addr buffer 

将 PE 文件名插入编辑控件。 **AppendText **通过传递一个 **EM\_REPLACESEL **消息以通知向编辑控件添加文本。然后它又向编辑控件发送一个设置了 wParam=-1 和 lParam=0 的**EM\_SETSEL** 消息，使光标定位到文本末。

   mov edi,pNTHdr   
   assume edi:ptr IMAGE\_NT\_HEADERS   
   mov edi, \[edi\].OptionalHeader.DataDirectory\[sizeof IMAGE\_DATA\_DIRECTORY\].VirtualAddress 

获取 import symbols 的 RVA。edi 起初指向 PE header，以此我们可以定位到数据目录数组的第二个数组元素来得到虚拟地址值。

   invoke RVAToOffset,pMapping,edi   
   mov edi,eax   
   add edi,pMapping 

这儿对 PE 编程初学者来说可能有点困难。在 PE 文件中大多数地址多是 RVAs 而 **RVAs****只有当****PE****文件被****PE****装载器装入内存后才有意义。** 本例中，我们直接将文件映射到内存而不是通过 PE 装载器载入，因此我们不能直接使用那些 RVAs。必须先将那些 RVAs 转换成文件偏移量，RVAToOffset 函数就起到这个作用。 这里不准备详细分析。指出的是，它还将给定的 RVA 和 PE 文件所有节的始末 RVA 作比较（检验 RVA 的有效性），然后通过**IMAGE\_SECTION\_HEADER** 结构中的**PointerToRawData**域（当然是所在节的那个**PointerToRawData**域啦）将 RVA 转换成文件偏移量。  
函数使用需要传递两个参数: 内存映射文件指针和所要转换的 RVA。eax 里返回文件偏移量。上面代码中，我们必须将文件偏移量加上内存映射文件指针以转换成虚拟地址。是不是有点复杂? :)

   assume edi:ptr IMAGE\_IMPORT\_DESCRIPTOR   
   .while !(\[edi\].OriginalFirstThunk==0 && \[edi\].TimeDateStamp==0 && \[edi\].ForwarderChain==0 && \[edi\].Name1==0 && \[edi\].FirstThunk==0) 

edi 现在指向第一个 **IMAGE\_IMPORT\_DESCRIPTOR** 结构。接下来我们遍历整个结构数组直到遇上一个全 0 结构，这就是数组末尾了。

     invoke AppendText,hDlg,addr ImportDescriptor  
     invoke RVAToOffset,pMapping, \[edi\].Name1   
     mov edx,eax   
     add edx,pMapping 

我们要显示当前 **IMAGE\_IMPORT\_DESCRIPTOR** 结构的值。Name1 不同于其他结构成员，它含有指向相关 dll 名的 RVA。因此必须先将其转换成虚拟地址。

     invoke wsprintf, addr temp, addr IDTemplate, \[edi\].OriginalFirstThunk,\[edi\].TimeDateStamp,\[edi\].ForwarderChain,edx,\[edi\].FirstThunk      invoke AppendText,hDlg,addr temp 

显示当前 **IMAGE\_IMPORT\_DESCRIPTOR** 结构的值。

     .if \[edi\].OriginalFirstThunk==0   
        mov esi,\[edi\].FirstThunk   
     .else   
        mov esi,\[edi\].OriginalFirstThunk   
     .endif 

接下来准备遍历 **IMAGE\_THUNK\_DATA** 数组。通常我们会选择**OriginalFirstThunk**指向的那个数组，不过，如果某些连接器错误地将**OriginalFirstThunk** 置 0，这可以通过检查**OriginalFirstThunk**值是否为 0 判断。这样的话，只要选择**FirstThunk**指向的数组了。

     invoke RVAToOffset,pMapping,esi   
     add eax,pMapping   
     mov esi,eax 

同样的，**OriginalFirstThunk**/**FirstThunk**值是一个 RVA。必须将其转换为虚拟地址。

     invoke AppendText,hDlg,addr NameHeader  
     .while dword ptr \[esi\]!=0 

现在我们准备遍历**IMAGE\_THUNK\_DATAs** 数组以查找该 DLL 引入的函数名，直到遇上全 0 项。

       test dword ptr \[esi\],IMAGE\_ORDINAL\_FLAG32   
       jnz ImportByOrdinal 

第一件事是校验**IMAGE\_THUNK\_DATA** 是否含有**IMAGE\_ORDINAL\_FLAG32**标记。检查**IMAGE\_THUNK\_DATA** 的 MSB 是否为 1，如果是 1，则函数是通过序数引出的，所以不需要更进一步处理了。直接从 **IMAGE\_THUNK\_DATA** 提取低字节获得序数，然后是下一个**IMAGE\_THUNK\_DATA** 双字。

       invoke RVAToOffset,pMapping,dword ptr \[esi\]   
       mov edx,eax   
       add edx,pMapping   
       assume edx:ptr IMAGE\_IMPORT\_BY\_NAME 

如果**IMAGE\_THUNK\_DATA** 的 MSB 是 0，那么它包含了 IMAGE\_IMPORT\_BY\_NAME 结构的 RVA。需要先转换为虚拟地址。

       mov cx, \[edx\].Hint   
       movzx ecx,cx   
       invoke wsprintf,addr temp,addr NameTemplate,ecx,addr \[edx\].Name1   
       jmp ShowTheText 

Hint 是字类型，所以先转换为双字后再传递给 wsprintf，然后我们将 hint 和函数名都显示到编辑控件中。

ImportByOrdinal:   
       mov edx,dword ptr \[esi\]   
       and edx,0FFFFh   
       invoke wsprintf,addr temp,addr OrdinalTemplate,edx 

在仅用序数引出函数的情况中，先清空高字再显示序数。

ShowTheText:   
       invoke AppendText,hDlg,addr temp   
       add esi,4 

在编辑控件中插入相应的函数名/序数后，跳转到下个 **IMAGE\_THUNK\_DATA**。

    .endw   
    add edi,sizeof IMAGE\_IMPORT\_DESCRIPTOR 

处理完当前**IMAGE\_THUNK\_DATA** 数组里的所有双字，跳转到下个**IMAGE\_IMPORT\_DESCRIPTOR** 开始处理其他 DLLs 的引入函数了。

### 附录:

让我们再来讨论一下 bound import。当 PE 装载器装入 PE 文件时，检查引入表并将相关 DLLs 映射到进程地址空间。然后象我们这样遍历**IMAGE\_THUNK\_DATA** 数组并用引入函数的真实地址替换**IMAGE\_THUNK\_DATAs** 值。这一步需要很多时间。如果程序员能事先正确预测函数地址，PE 装载器就不用每次装入 PE 文件时都去修正**IMAGE\_THUNK\_DATAs** 值了。Bound import 就是这种思想的产物。  
为了方便实现，Microsoft 出品的类似 Visual Studio 的编译器多提供了**bind.exe**这样的工具，由它检查 PE 文件的引入表并用引入函数的真实地址替换**IMAGE\_THUNK\_DATA** 值。当文件装入时，PE 装载器必定检查地址的有效性，如果 DLL 版本不同于 PE 文件存放的相关信息，或则 DLLs 需要重定位，那么装载器认为原先计算的地址是无效的，它必定遍历**OriginalFirstThunk**指向的数组以获取引入函数新地址。  
Bound import 在本课中并非很重要，我们确省就是用到了 OriginalFirstThunk。

**翻译：****iamgufeng \[**[**Iczelion's Win32 Assembly Homepage**](http://win32asm.cjb.net/)**\]****\[**[**LuoYunBin's Win32 ASM Page**](http://asm.yeah.net)**\]**

 

 4dd466d0e1e0cc123afbe504da2462ca