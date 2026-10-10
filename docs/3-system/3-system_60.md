

# 分析 zeroadd 算法，再造 PE 节增加器

> 原文：[`www.pediy.com/kssd/pediy10/81139.html`](https://www.pediy.com/kssd/pediy10/81139.html)

【文章标题】: 分析 zeroadd 算法，再造 PE 节增加器  
【文章作者】: zhujian  
【作者邮箱】: zhujian198@gmail.com  
【软件名称】: zeroadd  
【下载地址】: 自己搜索下载  
【编写语言】: 汇编  
【操作平台】: win x86  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
      在看雪注册这么多年，一直潜水，实在惭愧。今天在修改一个软件时候，由于 EXE 剩余间隙太少想使用 zeroadd 增加个节。  
  刚下载下来就被杀毒软件给杀了。做了个免杀，虽然可以用了。但心里不爽，再加上这两天在学习 PE 结构，前两天刚用汇编  
  写了个 PE 查看器，心里就有了干脆自己写一个 zeroadd。虽然知道增加 PE 增加节的原理，但是还是有些盲点。于是干脆开启神  
  器 OD，分析 zeroadd 的算法。  
  分析过程如下：  
  首先来到事件处理函数  
    
  ;如果选择了备份，那么就复制一份做为备份  
  0040110A  |.  E8 3B040000   call    <jmp.&USER32.IsDlgButtonChecked> ; \\IsDlgButtonChecked  
  0040110F  |.  83F8 01       cmp     eax, 1  
  00401112  |.  75 38         jnz     short 0040114C  
  00401114  |.  68 20304000   push    00403020                         ; /String2 = "C:\\Documents and Settings\\zhujian\\Desktop\\zeroadd 1.0\\prjVcSp6.exe"  
  00401119  |.  68 85304000   push    00403085                         ; |String1 = zeroadd.00403085  
  0040111E  |.  E8 81040000   call    <jmp.&KERNEL32.lstrcpyA>         ; \\lstrcpyA  
  00401123  |.  68 1B314000   push    0040311B                         ; /StringToAdd = ".bak"  
  00401128  |.  68 85304000   push    00403085                         ; |ConcatString = ""  
  0040112D  |.  E8 6C040000   call    <jmp.&KERNEL32.lstrcatA>         ; \\lstrcatA  
  00401132  |.  6A 00         push    0                                ; /FailIfExists = FALSE  
  00401134  |.  68 85304000   push    00403085                         ; |NewFileName = ""  
  00401139  |.  68 20304000   push    00403020                         ; |ExistingFileName = "C:\\Documents and Settings\\zhujian\\Desktop\\zeroadd 1.0\\prjVcSp6.exe"  
  0040113E  |.  E8 1F040000   call    <jmp.&KERNEL32.CopyFileA>        ; \\CopyFileA  
  00401143  |.  83F8 00       cmp     eax, 0  
    
  ; 打开要增加节的原始 exe 文件  
  00401166  |.  6A 00         push    0                                ; /hTemplateFile = NULL  
  00401168  |.  68 80000000   push    80                               ; |Attributes = NORMAL  
  0040116D  |.  6A 03         push    3                                ; |Mode = OPEN\_EXISTING  
  0040116F  |.  6A 00         push    0                                ; |pSecurity = NULL  
  00401171  |.  6A 00         push    0                                ; |ShareMode = 0  
  00401173  |.  68 000000C0   push    C0000000                         ; |Access = GENERIC\_READ|GENERIC\_WRITE  
  00401178  |.  68 20304000   push    00403020                         ; |FileName = "C:\\Documents and Settings\\zhujian\\Desktop\\zeroadd 1.0\\prjVcSp6.exe"  
  0040117D  |.  E8 E6030000   call    <jmp.&KERNEL32.CreateFileA>      ; \\CreateFileA  
  0040118B  |.  A3 7C354000   mov     dword ptr \[40357C\], eax   ;保存句柄  
    
  ;将界面上输入的节大小转换成 16 进制数字  
  00401190  |.  68 08304000   push    00403008                         ;  ASCII "200"  
  00401195  |.  E8 13030000   call    004014AD   ;转换字符为 16 进制数字   
  0040119A  |.  A3 98354000   mov     dword ptr \[403598\], eax   ;保存  200h   
    
  ;获得 exe 文件原始大小  
  0040119F  |.  6A 00         push    0                                ; /pFileSizeHigh = NULL  
  004011A1  |.  FF35 7C354000 push    dword ptr \[40357C\]               ; |hFile = 000001AC (window)  
  004011A7  |.  E8 D4030000   call    <jmp.&KERNEL32.GetFileSize>      ; \\GetFileSize  
    
  ;原始大小上 + 欲增加大小 + 0x2710    为什么是 2710,估计是随便增大点吧  
  004011AC  |.  0305 98354000 add     eax, dword ptr \[403598\]   ; 文件大小 +　２００H  
  004011B2  |.  05 10270000   add     eax, 2710  ;再加上   2710h  why???  
    
  ;创建一个文件镜像 大小为上面计算的大小  
  004011B7  |.  6A 00         push    0                                ; /MapName = NULL  
  004011B9  |.  50            push    eax                              ; |MaximumSizeLow  
  004011BA  |.  6A 00         push    0                                ; |MaximumSizeHigh = 0  
  004011BC  |.  6A 04         push    4                                ; |Protection = PAGE\_READWRITE  
  004011BE  |.  6A 00         push    0                                ; |pSecurity = NULL  
  004011C0  |.  FF35 7C354000 push    dword ptr \[40357C\]               ; |hFile = 000001AC (window)  
  004011C6  |.  E8 A3030000   call    <jmp.&KERNEL32.CreateFileMapping>; \\CreateFileMappingA  
    
  ;保存句柄 ，  然后 MAPVIEW  
  004011D3  |.  A3 80354000   mov     dword ptr \[403580\], eax  
  004011D8  |.  6A 00         push    0                                ; /MapSize = 0  
  004011DA  |.  6A 00         push    0                                ; |OffsetLow = 0  
  004011DC  |.  6A 00         push    0                                ; |OffsetHigh = 0  
  004011DE  |.  6A 02         push    2                                ; |AccessMode = FILE\_MAP\_WRITE  
  004011E0  |.  50            push    eax                              ; |hMapObject  
  004011E1  |.  E8 A6030000   call    <jmp.&KERNEL32.MapViewOfFile>    ; \\MapViewOfFile  
    
  ;保存上面 MapView 的句柄  
  004011EE  |.  A3 84354000   mov     dword ptr \[403584\], eax  ;保存句柄  
    
  ;判断 PE 有效？  
  004011F3  |.  66:8138 4D5A  cmp     word ptr \[eax\], 5A4D  ;判断 DOS 标识  
  004011FE  |.  0FB778 3C     movzx   edi, word ptr \[eax+3C\]  
  00401202  |.  033D 84354000 add     edi, dword ptr \[403584\]  
  00401208  |.  813F 50450000 cmp     dword ptr \[edi\], 4550  ;判断 NT 标识  
    
  ;读取 PE 相关信息  节的数目，可选头大小  
  00401214  |.  893D 88354000 mov     dword ptr \[403588\], edi  ;edi NT 头起始地址  
  0040121A  |.  0FB74F 06     movzx   ecx, word ptr \[edi+6\]  ;节数  
  0040121E  |.  66:037F 14    add     di, word ptr \[edi+14\]  ；可选头的大小  
    
  ;指针移到节表的开始处    di(nt 起始) + 可选头大小 + x018 (PE 标识 + nt 文件头大小 正好是 0x18)  
  00401222  |.  83C7 18       add     edi, 18  ；加 18H （PE 标识+ NT 文件头大小）  
    
  ;循环节个数次，移动到节表末尾  
  00401225  |> /83C7 28       /add     edi, 28  
  00401228  |.^\\E2 FB         \\loopd   short 00401225  
    
  ;判断节表区后面 9 个 DWORD 是否为空，有足够空间不？？  
  0040122A  |.  B9 0A000000   mov     ecx, 0A  
  0040122F  |.  BA 28000000   mov     edx, 28  
  00401234  |>  833C3A 00     /cmp     dword ptr \[edx+edi\], 0  
  00401238  |.  0F85 92010000 |jnz     004013D0  
  0040123E  |.  83C2 04       |add     edx, 4  
  00401241  |.^ E2 F1         \\loopd   short 00401234  
    
    
    
  00401243  |.  8B35 88354000 mov     esi, dword ptr \[403588\] ；PE 头地址  
  00401249  |.  66:FF46 06    inc     word ptr \[esi+6\]  ;节数目 +１  
  0040124D  |.  B9 01000000   mov     ecx, 1　；循环变量　至　　１  
  00401252  |.  837E 38 00    cmp     dword ptr \[esi+38\], 0　;节对齐  1000H  等于 O 吗  
  00401256  |.  75 08         jnz     short 00401260  
  00401258  |.  837E 3C 00    cmp     dword ptr \[esi+3C\], 0  ;文件对齐     等于 0 吗  
  00401260  |> \\83EF 28       sub     edi, 28   ;获得最后一个节的起始地址  
  00401263  |.  8B47 0C       mov     eax, dword ptr \[edi+C\] ;节内存偏移地址  
  00401266  |.  0347 08       add     eax, dword ptr \[edi+8\] ;加上 虚拟大小 （数据实际大小）  
  0040126B  |.  FF76 38       push    dword ptr \[esi+38\] ;节对齐力度 1000H   
  0040126E  |.  50            push    eax  
  0040126F  |.  E8 A6020000   call    0040151A  
    
  ；计算对齐后的地址  
  0040126B  |.  FF76 38       push    dword ptr \[esi+38\]  ;4000H  
  0040126E  |.  50            push    eax  
  0040126F  |.  E8 A6020000   call    0040151A  ;  5000H  
  -----------------40151A 计算对齐值函数----------------------  
    
      0040151A  /$  55            push    ebp  
      0040151B  |.  8BEC          mov     ebp, esp  
      0040151D  |.  53            push    ebx  
      0040151E  |.  52            push    edx  
      0040151F  |.  8B45 08       mov     eax, dword ptr \[ebp+8\]  
      00401522  |.  8B5D 0C       mov     ebx, dword ptr \[ebp+C\]  
      00401525  |.  33D2          xor     edx, edx  
      00401527  |>  50            /push    eax  
      00401528  |.  F7F3          |div     ebx  
      0040152A  |.  85D2          |test    edx, edx  
      0040152C  |.  58            |pop     eax  
      0040152D  |.  74 03         |je      short 00401532  
      0040152F  |.  40            |inc     eax  
      00401530  |.^ EB F5         \\jmp     short 00401527  
      00401532  |>  5A            pop     edx  
      00401533  |.  5B            pop     ebx  
      00401534  |.  C9            leave  
      00401535  \\.  C2 0800       retn    8  
  -----------------------------------------------------------  
    
  00401279  |.  8B47 14       mov     eax, dword ptr \[edi+14\]  ;节文件大小  
  0040127C  |.  0347 10       add     eax, dword ptr \[edi+10\]  ;节文件偏移  
    
  00401281  |.  FF76 3C       push    dword ptr \[esi+3C\] ;文件对齐力度  
  00401284  |.  50            push    eax    ;上面计算出来的和   
  00401285  |.  E8 90020000   call    0040151A   ;  5000H  
    
    
  ;修改 IMAGESZIE  
    
  004012AD  |> \\0146 50       add     dword ptr \[esi+50\], eax  ;修改 IMAGESIZE 大小   
  004012B0  |.  83C7 28       add     edi, 28  ；后移到最后一个节表  
    
  ；给新增加的节的第一个名字字段赋值  
  004012B3  |.  57            push    edi  
  004012B4  |.  8D35 00304000 lea     esi, dword ptr \[403000\]  
  004012BA  |.  B9 08000000   mov     ecx, 8  
  004012BF  |.  F3:A4         rep     movs byte ptr es:\[edi\], byte ptr \[esi\]  
  004012C1  |.  5F            pop     edi  
    
  ; 给新节表的各个字段赋值  
  004012C2  |.  A1 98354000   mov     eax, dword ptr \[403598\]  
  004012C7  |.  8947 08       mov     dword ptr \[edi+8\], eax  
  004012CA  |.  A1 90354000   mov     eax, dword ptr \[403590\]  
  004012CF  |.  8947 0C       mov     dword ptr \[edi+C\], eax  
  004012D2  |.  A1 98354000   mov     eax, dword ptr \[403598\]  
  004012D7  |.  8947 10       mov     dword ptr \[edi+10\], eax  
  004012DA  |.  A1 94354000   mov     eax, dword ptr \[403594\]  
  004012DF  |.  8947 14       mov     dword ptr \[edi+14\], eax  
  004012E2  |.  C747 24 20000>mov     dword ptr \[edi+24\], E0000020  
    
    
  004012E9  |.  8B3D 84354000 mov     edi, dword ptr \[403584\]  ;最后一节表起始  
  004012EF  |.  033D 94354000 add     edi, dword ptr \[403594\] ;  
  004012F5  |.  8B0D 98354000 mov     ecx, dword ptr \[403598\]  
  004012FB  |.  32C0          xor     al, al  
  004012FD  |.  F3:AA         rep     stos byte ptr es:\[edi\] ;增加的新节全部清零  
    
  ;后面就是写入文件，关闭文件，销毁句柄等等了。基本算法就到这里了。  
    
  总结,增加新节做了如下几个步骤：  
  1、基于镜像方式把 PE 载入内存，方便操作。当然也可以直接写文件  
  2、读取节的个数，判断节表后面是否有足够空间增加新节，如果没空间就 OVER  
  3、如果有空间就增加新节表，填写节表相关项。主要内存对齐和文件对齐力度  
  4、重新计算 IMAGESIZE 大小，并修改  
  5、新节内存区域全部清零  
  6、重新把输入写回硬盘  
    
  下面是 C 语言实现代码：  
  /\*  
  \*  作者:zhujian  
  \*  描述:为 PE 文件增加新节  
  \*  日期:2009/1/30  
  \*/  
  #include <windows.h>  
  #include <string.h>  
  #include <stdlib.h>  
  #include "resource.h"  
  char szFileName\[MAX\_PATH\] = {0};  //界面上输入的文件名  
  char szSectionName\[MAX\_PATH\] = {0}; //界面上输入的要增加的节的名字  
  char szSectionSize\[MAX\_PATH\] = {0}; //界面上输入的节的大小字符串表示  
  char szBakFileName\[MAX\_PATH\] = {0}; //备份文件的文件名  
  DWORD dwOrgFileHandle = 0; //原始文件的句柄  
  DWORD dwSectionSize = 0; // 界面上输入的要增加的节的大小 数字表示  
  DWORD dwFileSize = 0; //文件大小  
  DWORD dwFileMap = 0;//MapFile 的句柄  
  DWORD dwMapView = 0;//Map view 句柄  
    
  DWORD dwSecNum = 0;//原来节的数目  
  int i;  
    
    
    
  char szTemp\[50\] = {0};//将输入的节大小拼凑成 16 进制字符串的一个临时变量  
  //-------  
  PIMAGE\_DOS\_HEADER pDosH = NULL;  
  PIMAGE\_NT\_HEADERS pNtH = NULL;  
  PIMAGE\_SECTION\_HEADER pFirstSecH= NULL; //节表第一项的首地址  
  PIMAGE\_SECTION\_HEADER pLastSecH= NULL; //节表最后一项的首地址  
  PIMAGE\_SECTION\_HEADER pAddSecH= NULL; //新增加节表项的首地址  
    
  HINSTANCE mhInstance = NULL;  
  HICON mhIcon = NULL;  
    
  DWORD dwNewSecAlign = 0; //新增节的内存对齐  
  DWORD dwNewFileAlign = 0;//新增加节的文件对齐  
    
  DWORD dwOkFile = 0; //最后生成的文件的句柄  
     
    
  DWORD dwR = 0;  
  int translat(char c)  
  {  
    if(c<='9'&&c>='0') return c-'0';  
    if(c>='a' && c<='f') return c-87;  
    if(c>='A' && c<='F') return c-55;  
    return -1;  
  }  
  //16 进制字符串转 10 进制数字  
  int Htoi(char \*str)  
  {  
    int i,n=0,stat;  
    int length=strlen(str);  
    if(length==0) return 0;  
      
    for(i=0;i<length;i++)  
    {  
      stat=translat(str\[i\]);  
      if(stat>=0) n=n\*16+stat;  
    }  
    return n;  
  }   
  //计算对齐后的地址  
  DWORD getAlign(DWORD addr,DWORD base)  
  {  
    int j = 0;  
    for (j = addr;j % base != 0;j++)  
    {  
    
    }  
    return j;  
  }  
    
    
  INT\_PTR CALLBACK DialogProc(  
                HWND hwndDlg,  // handle to dialog box  
                UINT uMsg,     // message  
                WPARAM wParam, // first message parameter  
                LPARAM lParam  // second message parameter  
  )  
  {  
    switch(uMsg)  
    {  
    case WM\_CLOSE:  
       EndDialog(hwndDlg,NULL);  
       return TRUE;  
    case WM\_INITDIALOG:  
      mhIcon = LoadIcon(mhInstance,IDI\_ICON1);  
      SendMessage(hwndDlg,WM\_SETICON,ICON\_BIG,mhIcon);  
      return TRUE;  
    case WM\_COMMAND:  
       if (wParam == IDC\_BUTTON\_EXIT)  
       {  
         EndDialog(hwndDlg,NULL);  
           return TRUE;  
       }  
       else if (wParam == IDC\_BUTTON\_SELFILE)  
       {  
         OPENFILENAME stOFN = {0};  
         stOFN.lStructSize = sizeof stOFN;  
         stOFN.lpstrFilter = "Executable Files(\*.exe,\*.dll)\\0\*.exe\\0\\All File(\*.\*)\\0\*.\*\\0";  
         stOFN.nMaxFile = MAX\_PATH;  
         stOFN.lpstrFile = szFileName;  
  
         stOFN.hwndOwner = hwndDlg;  
         stOFN.Flags = OFN\_FILEMUSTEXIST | OFN\_PATHMUSTEXIST;  
         if (GetOpenFileName(&stOFN))  
         {  
           SetDlgItemText(hwndDlg,IDC\_EDIT\_SELFILE,szFileName);  
         }         
         return TRUE;  
       }  
       else if (wParam == IDC\_BUTTON\_ADD)  
       {  
         GetDlgItemText(hwndDlg,IDC\_EDIT\_NAME,szSectionName,MAX\_PATH);  
         GetDlgItemText(hwndDlg,IDC\_EDIT\_SIZE,szSectionSize,MAX\_PATH);  
    
         if (strlen(szSectionSize)==0)  
         {  
           MessageBox(hwndDlg,"请输入新节的名字","提示",MB\_OK);  
           return TRUE;  
         }  
         if (strlen(szSectionSize)==0)  
         {  
           MessageBox(hwndDlg,"请输入新节的大小","提示",MB\_OK);  
           return TRUE;  
         }  
    
         //备份原程序  
         if (IsDlgButtonChecked(hwndDlg,IDC\_CHECK\_BAK))  
         {  
          strcpy(szBakFileName,szFileName);  
          strcat(szBakFileName,".bak");  
          CopyFile(szFileName,szBakFileName,FALSE);  
    
         }  
    
         //打开原 EXE 准备增加节  
         dwOrgFileHandle = CreateFile(szFileName,GENERIC\_READ|GENERIC\_WRITE,FILE\_SHARE\_READ,NULL,OPEN\_EXISTING,FILE\_ATTRIBUTE\_NORMAL,NULL);  
         strcat(szTemp,"0x");  
         strcat(szTemp,szSectionSize);  
         dwSectionSize = Htoi(szTemp);//字符转数字  
         //获得文件大小  
         dwFileSize = GetFileSize(dwOrgFileHandle,NULL);  
         dwFileSize += dwSectionSize;  
         dwFileSize += 0x00002710;//逆向 addzero 时候得到的数字，反正就是加大点  
         dwFileMap = CreateFileMapping(dwOrgFileHandle,NULL,PAGE\_READWRITE,0,dwFileSize,NULL);  
         dwMapView = MapViewOfFile(dwFileMap,FILE\_MAP\_WRITE|FILE\_MAP\_READ,NULL,NULL,NULL);  
           
         pDosH = (PIMAGE\_DOS\_HEADER)dwMapView;  
         pNtH = (PBYTE)(dwMapView)+(pDosH->e\_lfanew);  
         if(IMAGE\_DOS\_SIGNATURE == pDosH->e\_magic)  
         {  
            
          if (IMAGE\_NT\_SIGNATURE == pNtH->Signature)     
          {                    // 增加节处理开始  
                
             dwSecNum = pNtH->FileHeader.NumberOfSections;  
             //节表开始处  
             pFirstSecH =(PBYTE)pNtH+(pNtH->FileHeader.SizeOfOptionalHeader + sizeof(IMAGE\_FILE\_HEADER) + 4);  
             //将要增加节的其实地址，它在最后一个节的后面  
             pAddSecH = (PBYTE)(pFirstSecH) + (sizeof(IMAGE\_SECTION\_HEADER) \* dwSecNum);  
               
             //判断最后一个节后面是否有多余空间来增加节表  
             for (i = 0;i < 10;i++)  
             {  
               if (\*((PDWORD)(pAddSecH)+i)!=0)  
               {  
                 break;  
               }  
             }  
             if (10 == i)  
             {//表示有空间增加新节表  
                 
               pNtH->FileHeader.NumberOfSections++;//节表数加 1  
               //获得最后一个节  
               pLastSecH = ((PBYTE)(pAddSecH) - sizeof(IMAGE\_SECTION\_HEADER));  
               if (pNtH->OptionalHeader.FileAlignment != 0 && pNtH->OptionalHeader.SectionAlignment != 0)  
               {  
                 //节对齐处理  
                  dwNewSecAlign = getAlign(pLastSecH->VirtualAddress + pLastSecH->Misc.VirtualSize,pNtH->OptionalHeader.SectionAlignment);  
    
                 //文件对齐处理  
                  
                dwNewFileAlign = getAlign(pLastSecH->PointerToRawData+pLastSecH->SizeOfRawData,pNtH->OptionalHeader.FileAlignment);  
                //设置新增加节表的各个项  
                pNtH->OptionalHeader.SizeOfImage += getAlign(dwSectionSize,pNtH->OptionalHeader.SectionAlignment);  
                memcpy(pAddSecH->Name,szSectionName,8); //name  
                pAddSecH->VirtualAddress = dwNewSecAlign;  
                pAddSecH->Misc.VirtualSize = dwSectionSize;  
    
                pAddSecH->PointerToRawData = dwNewFileAlign;  
                pAddSecH->SizeOfRawData = getAlign(dwSectionSize,pNtH->OptionalHeader.FileAlignment);  
                pAddSecH->Characteristics = 0xE0000020; //Characteristics  
                //最后一个节内存全部清零  
                RtlZeroMemory(((PBYTE)(pDosH) + pAddSecH->PointerToRawData),getAlign(dwSectionSize,pNtH->OptionalHeader.FileAlignment));  
    
                //写入硬盘  
                dwOkFile = CreateFile("swap.pe",GENERIC\_READ|GENERIC\_WRITE,FILE\_SHARE\_READ,NULL,CREATE\_ALWAYS,FILE\_ATTRIBUTE\_NORMAL,NULL);  
                WriteFile(dwOkFile,dwMapView,pAddSecH->PointerToRawData + pAddSecH->SizeOfRawData,&dwR,NULL);  
                CopyFile("swap.pe",szFileName,FALSE);  
                CloseHandle(dwOkFile);  
                DeleteFile("swap.pe");  
                MessageBox(hwndDlg,"增加新节成功","恭喜",MB\_OK);  
               }  
               else  
               {  
                 MessageBox(hwndDlg,"PE 文件对齐或内存对齐为零","失败",MB\_ICONINFORMATION);  
               }  
             }  
             else  
             {  
               MessageBox(hwndDlg,"没有足够空间增加节表","提示",MB\_ICONINFORMATION);  
             }  
          }  
          else  
          {  
             MessageBox(hwndDlg,"无效 PE 文件,NT 标识错误","提示",MB\_ICONINFORMATION);  
          }  
             
         }  
         else  
         {  
           MessageBox(hwndDlg,"无效 PE 文件","提示",MB\_ICONINFORMATION);  
             
         }  
           
         UnmapViewOfFile(dwMapView);  
         CloseHandle(dwFileMap);  
         CloseHandle(dwOrgFileHandle);  
         return TRUE;  
       }  
    }  
    return FALSE;  
  }  
    
  int WINAPI WinMain(  
    HINSTANCE hInstance,      // handle to current instance  
    HINSTANCE hPrevInstance,  // handle to previous instance  
    LPSTR lpCmdLine,          // command line  
    int nCmdShow              // show state  
  )  
  {  
    mhInstance = hInstance;  
    DialogBoxParam(hInstance,IDD\_DIALOG\_ADDSECTION,NULL,DialogProc,NULL);  
    ExitProcess(NULL);  
    return 0;  
  }  
    
  小弟潜水多年的菜鸟难免有不足之处，代码只是实现了基本功能，有很多地方没做容错判断。  
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2009 年 01 月 31 日 14:49:50