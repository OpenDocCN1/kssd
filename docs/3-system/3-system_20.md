

# PE 教程 5: Section Table（节表）

> 原文：[`www.pediy.com/kssd/tutorial/chap8-1-5.htm`](https://www.pediy.com/kssd/tutorial/chap8-1-5.htm)

 

# PE 教程 5: Section Table（节表）

请下载 范例。

### 理论:

到本课为止，我们已经学了许多关于 DOS header 和 PE header 的知识。接下来就该轮到 section table（节表）了。节表其实就是紧挨着 PE header 的一结构数组。该数组成员的数目由 file header (**IMAGE\_FILE\_HEADER**) 结构中 **NumberOfSections** 域的域值来决定。节表结构又命名为 **IMAGE\_SECTION\_HEADER**。

**IMAGE\_SIZEOF\_SHORT\_NAME equ 8 **

**IMAGE\_SECTION\_HEADER STRUCT   
   Name1 db IMAGE\_SIZEOF\_SHORT\_NAME dup(?)   
   union Misc   
      PhysicalAddress dd ?   
      VirtualSize dd ?   
   ends   
   VirtualAddress dd ?   
   SizeOfRawData dd ?   
   PointerToRawData dd ?   
   PointerToRelocations dd ?   
   PointerToLinenumbers dd ? ****哦  
****   NumberOfRelocations dw ?   
   NumberOfLinenumbers dw ?   
   Characteristics dd ?   
IMAGE\_SECTION\_HEADER ENDS **

同样，不是所有成员都是很有用的，我们只关心那些真正重要的。

| **Field** || Meanings |
| --- | --- |
| **Name1** || 事实上本域的名称是"name"，只是"name"已被 MASM 用作关键字，所以我们只能用"Name1"代替。这儿的节名长不超过 8 字节。记住节名仅仅是个标记而已，我们选择任何名字甚至空着也行，注意这里不用 null 结束。命名**不是**一个 ASCIIZ 字符串，所以不用 null 结尾。 |
| **VirtualAddress** || 本节的 RVA（相对虚拟地址）。PE 装载器将节映射至内存时会读取本值，因此如果域值是 1000h，而 PE 文件装在地址 400000h 处，那么本节就被载到 401000h。 |
| **SizeOfRawData** || 经过文件对齐处理后节尺寸，PE 装载器提取本域值了解需映射入内存的节字节数。（译者注: 假设一个文件的文件对齐尺寸是 0x200，如果前面的** VirtualSize**域指示本节长度是 0x388 字节，则本域值为 0x400，表示本节是 0x400 字节长）。 |
| **PointerToRawData** || 这是节基于文件的偏移量，PE 装载器通过本域值找到节数据在文件中的位置。 |
| **Characteristics** || 包含标记以指示节属性，比如节是否含有可执行代码、初始化数据、未初始数据，是否可写、可读等。 |

现在我们已知晓 **IMAGE\_SECTION\_HEADER** 结构，再来模拟一下 PE 装载器的工作吧:

1.  读取 **IMAGE\_FILE\_HEADER** 的 **NumberOfSections**域，知道文件的节数目。
2.  **SizeOfHeaders** 域值作为节表的文件偏移量，并以此定位节表。
3.  遍历整个结构数组检查各成员值。
4.  对于每个结构，我们读取**PointerToRawData**域值并定位到该文件偏移量。然后再读取**SizeOfRawData**域值来决定映射内存的字节数。将**VirtualAddress**域值加上**imageBase**域值等于节起始的虚拟地址。然后就准备把节映射进内存，并根据**Characteristics**域值设置属性。
5.  遍历整个数组，直至所有节都已处理完毕。

注意我们并没有使用节名: 这其实并不重要。

### 示例:

本例程打开一 PE 文件遍历其节表，并在列表框控件显示各节的信息。

.386   
.model flat,stdcall   
option casemap:none   
include \\masm32\\include\\windows.inc   
include \\masm32\\include\\kernel32.inc   
include \\masm32\\include\\comdlg32.inc   
include \\masm32\\include\\user32.inc   
include \\masm32\\include\\comctl32.inc   
includelib \\masm32\\lib\\comctl32.lib   
includelib \\masm32\\lib\\user32.lib   
includelib \\masm32\\lib\\kernel32.lib   
includelib \\masm32\\lib\\comdlg32.lib   
  
IDD\_SECTIONTABLE equ 104   
IDC\_SECTIONLIST equ 1001   
  
SEH struct 

  
PrevLink dd ? ; the address of the previous seh structure   
CurrentHandler dd ? ; the address of the new exception handler   
SafeOffset dd ? ; The offset where it's safe to continue execution   
PrevEsp dd ? ; the old value in esp   
PrevEbp dd ? ; The old value in ebp   
SEH ends   
  
.data   
AppName db "PE tutorial no.5",0   
ofn OPENFILENAME <>   
FilterString db "Executable Files (\*.exe, \*.dll)",0,"\*.exe;\*.dll",0   
             db "All Files",0,"\*.\*",0,0   
FileOpenError db "Cannot open the file for reading",0   
FileOpenMappingError db "Cannot open the file for memory mapping",0   
FileMappingError db "Cannot map the file into memory",0   
FileInValidPE db "This file is not a valid PE",0   
template db "%08lx",0   
SectionName db "Section",0   
VirtualSize db "V.Size",0   
VirtualAddress db "V.Address",0   
SizeOfRawData db "Raw Size",0   
RawOffset db "Raw Offset",0   
Characteristics db "Characteristics",0   
  
.data?   
hInstance dd ?   
buffer db 512 dup(?)   
hFile dd ?   
hMapping dd ?   
pMapping dd ?   
ValidPE dd ?   
NumberOfSections dd ?   
  
.code   
start proc   
LOCAL seh:SEH   
   invoke GetModuleHandle,NULL   
   mov hInstance,eax   
   mov ofn.lStructSize,SIZEOF ofn   
   mov ofn.lpstrFilter, OFFSET FilterString   
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
                  call ShowSectionInfo   
               .else   
                  invoke MessageBox, 0, addr FileInValidPE, addr AppName, MB\_OK+MB\_ICONINFORMATION   
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
  invoke ExitProcess, 0   
  invoke InitCommonControls   
start endp   
  
SEHHandler proc uses edx pExcept:DWORD,pFrame:DWORD,pContext:DWORD,pDispatch:DWORD   
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
  
DlgProc proc uses edi esi hDlg:DWORD, uMsg:DWORD, wParam:DWORD, lParam:DWORD   
   LOCAL lvc:LV\_COLUMN   
   LOCAL lvi:LV\_ITEM   
   .if uMsg==WM\_INITDIALOG   
      mov esi, lParam   
      mov lvc.imask,LVCF\_FMT or LVCF\_TEXT or LVCF\_WIDTH or LVCF\_SUBITEM   
      mov lvc.fmt,LVCFMT\_LEFT   
      mov lvc.lx,80   
      mov lvc.iSubItem,0   
      mov lvc.pszText,offset SectionName   
      invoke SendDlgItemMessage,hDlg,IDC\_SECTIONLIST,LVM\_INSERTCOLUMN,0,addr lvc inc lvc.iSubItem   
      mov lvc.fmt,LVCFMT\_RIGHT   
      mov lvc.pszText,offset VirtualSize   
      invoke SendDlgItemMessage,hDlg,IDC\_SECTIONLIST,LVM\_INSERTCOLUMN,1,addr lvc   
      inc lvc.iSubItem   
      mov lvc.pszText,offset VirtualAddress   
      invoke SendDlgItemMessage,hDlg,IDC\_SECTIONLIST,LVM\_INSERTCOLUMN,2,addr lvc   
      inc lvc.iSubItem   
      mov lvc.pszText,offset SizeOfRawData   
      invoke SendDlgItemMessage,hDlg,IDC\_SECTIONLIST,LVM\_INSERTCOLUMN,3,addr lvc   
      inc lvc.iSubItem   
      mov lvc.pszText,offset RawOffset   
      invoke SendDlgItemMessage,hDlg,IDC\_SECTIONLIST,LVM\_INSERTCOLUMN,4,addr lvc   
      inc lvc.iSubItem   
      mov lvc.pszText,offset Characteristics   
      invoke SendDlgItemMessage,hDlg,IDC\_SECTIONLIST,LVM\_INSERTCOLUMN,5,addr lvc   
      mov ax, NumberOfSections   
      movzx eax,ax   
      mov edi,eax         
      mov lvi.imask,LVIF\_TEXT   
      mov lvi.iItem,0   
      assume esi:ptr IMAGE\_SECTION\_HEADER   
      .while edi>0   
         mov lvi.iSubItem,0   
         invoke RtlZeroMemory,addr buffer,9   
         invoke lstrcpyn,addr buffer,addr \[esi\].Name1,8   
         lea eax,buffer   
         mov lvi.pszText,eax   
         invoke SendDlgItemMessage,hDlg,IDC\_SECTIONLIST,LVM\_INSERTITEM,0,addr lvi   
         invoke wsprintf,addr buffer,addr template,\[esi\].Misc.VirtualSize   
         lea eax,buffer   
         mov lvi.pszText,eax   
         inc lvi.iSubItem   
         invoke SendDlgItemMessage,hDlg,IDC\_SECTIONLIST,LVM\_SETITEM,0,addr lvi   
         invoke wsprintf,addr buffer,addr template,\[esi\].VirtualAddress   
         lea eax,buffer   
         mov lvi.pszText,eax   
         inc lvi.iSubItem   
         invoke SendDlgItemMessage,hDlg,IDC\_SECTIONLIST,LVM\_SETITEM,0,addr lvi   
         invoke wsprintf,addr buffer,addr template,\[esi\].SizeOfRawData   
         lea eax,buffer   
         mov lvi.pszText,eax   
         inc lvi.iSubItem   
         invoke SendDlgItemMessage,hDlg,IDC\_SECTIONLIST,LVM\_SETITEM,0,addr lvi   
         invoke wsprintf,addr buffer,addr template,\[esi\].PointerToRawData   
         lea eax,buffer   
         mov lvi.pszText,eax   
         inc lvi.iSubItem   
         invoke SendDlgItemMessage,hDlg,IDC\_SECTIONLIST,LVM\_SETITEM,0,addr lvi   
         invoke wsprintf,addr buffer,addr template,\[esi\].Characteristics   
         lea eax,buffer   
         mov lvi.pszText,eax   
         inc lvi.iSubItem   
         invoke SendDlgItemMessage,hDlg,IDC\_SECTIONLIST,LVM\_SETITEM,0,addr lvi   
         inc lvi.iItem   
         dec edi   
         add esi, sizeof IMAGE\_SECTION\_HEADER   
      .endw   
   .elseif   
      uMsg==WM\_CLOSE   
         invoke EndDialog,hDlg,NULL   
   .else   
      mov eax,FALSE   
      ret   
   .endif   
   mov eax,TRUE   
   ret   
DlgProc endp   
  
ShowSectionInfo proc uses edi   
   mov edi, pMapping   
   assume edi:ptr IMAGE\_DOS\_HEADER   
   add edi, \[edi\].e\_lfanew   
   assume edi:ptr IMAGE\_NT\_HEADERS   
   mov ax,\[edi\].FileHeader.NumberOfSections   
   movzx eax,ax   
   mov NumberOfSections,eax   
   add edi,sizeof IMAGE\_NT\_HEADERS   
   invoke DialogBoxParam, hInstance, IDD\_SECTIONTABLE,NULL, addr DlgProc, edi  
   ret   
ShowSectionInfo endp   
end start 

### 分析:

本例重用了 PE 教程 2 的代码，校验 PE 文件的有效性后，继续调用函数 ShowSectionInfo 显示各节信息。

ShowSectionInfo proc uses edi   
   mov edi, pMapping   
   assume edi:ptr IMAGE\_DOS\_HEADER   
   add edi, \[edi\].e\_lfanew  
   assume edi:ptr IMAGE\_NT\_HEADERS

我们将 edi 用作指向 PE 文件数据的指针。首先，将指向 DOS header 地址的 pMapping 赋给 edi，再加上**e\_lfanew**域值等于 PE header 的地址。

   mov ax,\[edi\].FileHeader.NumberOfSections  
   mov NumberOfSections,ax 

因为我们要遍历节表，所以必须先获取文件的节数目。这就得靠 file header 里的 NumberOfSections 域了，切记这是个 word 域。

   add edi,sizeof IMAGE\_NT\_HEADERS 

现在 edi 正指向 PE header 的起始地址，加上 PE header 结构大小后恰好指向节表了。

   invoke DialogBoxParam, hInstance, IDD\_SECTIONTABLE,NULL, addr DlgProc, edi

调用 **DialogBoxParam** 显示列表对话框，注意我们已将节表地址作为最后一个参数传递过去了，该值可从**WM\_INITDIALOG** 消息的 lParam 参数中提取。

在对话框过程里我们响应 WM\_INITDIALOG 消息，将 lParam 值 (节表地址)存入 esi，节数目赋给 edi 并设置列表控件。万事俱备后，进入循环将各节信息插入到列表控件中，这部分相当简单。

      .while edi>0   
         mov lvi.iSubItem,0 

字符串置入第一列。

         invoke RtlZeroMemory,addr buffer,9   
         invoke lstrcpyn,addr buffer,addr \[esi\].Name1,8   
         lea eax,buffer   
         mov lvi.pszText,eax 

要显示节名，当然要将其转换为 ASCIIZ 字符串先。

         invoke SendDlgItemMessage,hDlg,IDC\_SECTIONLIST,LVM\_INSERTITEM,0,addr lvi 

然后显示第一列。  
继续我们伟大的工程，显示完本节中最后一个欲呈现的值后，立马下一个结构。

         dec edi   
         add esi, sizeof IMAGE\_SECTION\_HEADER   
      .endw 

每处理完一节就递减 edi，然后将 esi 加上**IMAGE\_SECTION\_HEADER** 结构大小，使其指向下一个**IMAGE\_SECTION\_HEADER** 结构。

遍历节表的步骤:

1.  PE 文件有效性校验。
2.  定位到 PE header 的起始地址。
3.  从 file header 的**NumberOfSections**域获取节数。
4.  通过两种方法定位节表: **imageBase**+**SizeOfHeaders** 或者 PE header 的起始地址\+ PE header 结构大小。 (节表紧随 PE header)。如果不是使用文件映射的方法，可以用**SetFilePointer** 直接将文件指针定位到节表。节表的文件偏移量存放在 **SizeOfHeaders**域里。(**SizeOfHeaders** 是 **IMAGE\_OPTIONAL\_HEADER **的结构成员)
5.  处理每个 **IMAGE\_SECTION\_HEADER** 结构。

**翻译：****iamgufeng \[**[**Iczelion's Win32 Assembly Homepage**](http://win32asm.cjb.net/)**\]****\[**[**LuoYunBin's Win32 ASM Page**](http://asm.yeah.net)**\]**

 

 4dd466d0e1e0cc123afbe504da2462ca