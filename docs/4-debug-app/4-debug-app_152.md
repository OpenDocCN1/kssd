

# 让 Hex Workshop V6.0.1.4603 支持中文路径

> 原文：[`www.pediy.com/kssd/pediy11/115747.html`](https://www.pediy.com/kssd/pediy11/115747.html)

近日空闲的时候将 Hex Workshop v5 升成了 v6，结果发现它的右键菜单不支持中文路径，造成桌面上的文件无法用右键菜单打开，很是不方便，于是有了给 v6 动一下手术的想法。首先用上 IDA 和 OD 两个法宝，很快确定问题在以下两处位置的代码：  
  
获取并传递文件名的过程：  
.text:004862E0                 lea     edx, \[ebp+824h+var\_838\]  
.text:004862E3                 push    edx  
.text:004862E4                 lea     ecx, \[ebp+824h+var\_890\] ; 读取右键菜单通过临时文件传入的多字节文件名  
.text:004862E7                 call    ?ReadString@CStdioFile@@UAEHAAV?$CStringT@\_WV?$StrTraitMFC@\_WV?$ChTraitsCRT@\_W@ATL@@@@@ATL@@@Z ; CStdioFile::ReadString(ATL::CStringT<wchar\_t,StrTraitMFC<wchar\_t,ATL::ChTraitsCRT<wchar\_t>>> &)  
.text:004862EC                 cmp     eax, ebx  
.text:004862EE                 jz      short loc\_48632A  
.text:004862F0                 lea     ecx, \[ebp+824h+var\_838\]  
.text:004862F3                 call    sub\_476130  
.text:004862F8                 lea     ecx, \[ebp+824h+var\_838\]  
.text:004862FB                 call    sub\_476220  
.text:00486300                 mov     ecx, \[ebp+824h+var\_838\]  
.text:00486303                 lea     edx, \[ebp+824h+var\_824\]  
.text:00486306  
.text:00486306 loc\_486306:                             ; CODE XREF: sub\_4861D0+145 j  
.text:00486306                 movzx   eax, word ptr \[ecx\]  
.text:00486309                 mov     \[edx\], ax  
.text:0048630C                 add     ecx, 2  
.text:0048630F                 add     edx, 2  
.text:00486312                 cmp     ax, bx  
.text:00486315                 jnz     short loc\_486306  
.text:00486317                 mov     eax, \[ebp+824h+var\_83C\]  
.text:0048631A                 push    ebx  
.text:0048631B                 push    ebx  
.text:0048631C                 push    eax  
.text:0048631D                 lea     ecx, \[ebp+824h+var\_824\]  
.text:00486320                 push    ecx  
.text:00486321                 mov     ecx, edi  
.text:00486323                 call    sub\_484850      ; 传入 UNICODE 格式文件名到打开文件过程  
.text:00486328                 jmp     short loc\_4862E0  
  
打开文件的过程，由调用 CreateFileW 可以知道文件名必须是 UNICODE 格式。  
.text:00548222 loc\_548222:                             ; CODE XREF: sub\_548140+C4 j  
.text:00548222                 push    0               ; hTemplateFile  
.text:00548224                 push    80h             ; dwFlagsAndAttributes  
.text:00548229                 push    3               ; dwCreationDisposition  
.text:0054822B                 push    0               ; lpSecurityAttributes  
.text:0054822D                 push    3               ; dwShareMode  
.text:0054822F                 push    80000000h       ; dwDesiredAccess  
.text:00548234                 lea     ecx, \[esp+838h+FileName\]  
.text:00548238                 push    ecx             ; lpFileName  
.text:00548239                 call    ds:CreateFileW  ; 打开 UNICODE 文件名的文件  
.text:0054823F                 cmp     eax, 0FFFFFFFFh  
.text:00548242                 jz      short loc\_548257  
.text:00548244                 push    eax             ; hObject  
.text:00548245                 call    ds:CloseHandle  
.text:0054824B                 mov     eax, 1  
.text:00548250                 add     esp, 820h  
  
通过上面的两段代码，可以知道 Hex Workshop 使用了 CStdioFile::ReadString 读取临时文件传入的多字节文件路径。CStdioFile 读取 ANSI 文本数据时按 char 类型读取，在\_MSBC 下可以直接填充到 CString，在 UNICODE 环境下要先将 char 转换成宽字符 WCHAR，然后再填充到 CString，即一个汉字的两个 char 将变成两个 UNICODE 字符 WCHAR。因此 CStudioFile 在\_MSBC 环境下读取任何 ANSI 文本数据都没问题，在 UNICODE 环境下读取 ANSI 文本中的中文时就会显示乱码，但原来的汉字信息并没有丢失。知道了原因就好解决了，在 text 段发现还有 100 多字节的空间，于是添加下面的补丁代码：  
  
跳转到补丁代码：  
.text:004862E0 loc\_4862E0:                             ; CODE XREF: sub\_4861D0+158 j  
.text:004862E0                 lea     edx, \[ebp+824h+lpWideCharStr\]  
.text:004862E3                 push    edx  
.text:004862E4                 lea     ecx, \[ebp+824h+var\_890\]  
.text:004862E7                 call    ?ReadString@CStdioFile@@UAEHAAV?$CStringT@\_WV?$StrTraitMFC@\_WV?$ChTraitsCRT@\_W@ATL@@@@@ATL@@@Z ; CStdioFile::ReadString(ATL::CStringT<wchar\_t,StrTraitMFC<wchar\_t,ATL::ChTraitsCRT<wchar\_t>>> &)  
.text:004862EC                 cmp     eax, ebx  
.text:004862EE                 jmp     loc\_68AF88      ; 跳转到补丁代码  
.text:004862F3 ; ---------------------------------------------------------------------------  
.text:004862F3  
.text:004862F3 loc\_4862F3:                             ; CODE XREF: sub\_4861D0+204E2B j  
.text:004862F3                 call    sub\_476130  
.text:004862F8                 lea     ecx, \[ebp+824h+lpWideCharStr\]  
.text:004862FB                 call    sub\_476220  
.text:00486300                 mov     ecx, \[ebp+824h+lpWideCharStr\]  
.text:00486303                 lea     edx, \[ebp+824h+var\_824\]  
.text:00486306  
.text:00486306 loc\_486306:                             ; CODE XREF: sub\_4861D0+145 j  
.text:00486306                 movzx   eax, word ptr \[ecx\]  
.text:00486309                 mov     \[edx\], ax  
.text:0048630C                 add     ecx, 2  
.text:0048630F                 add     edx, 2  
.text:00486312                 cmp     ax, bx  
.text:00486315                 jnz     short loc\_486306  
.text:00486317                 mov     eax, \[ebp+824h+var\_83C\]  
.text:0048631A                 push    ebx  
.text:0048631B                 push    ebx  
.text:0048631C                 push    eax  
.text:0048631D                 lea     ecx, \[ebp+824h+var\_824\]  
.text:00486320                 push    ecx  
.text:00486321                 mov     ecx, edi  
.text:00486323                 call    sub\_484850  
  
补丁代码：  
.text:0068AF88 loc\_68AF88:                             ; CODE XREF: sub\_4861D0+11E j  
.text:0068AF88                 jz      loc\_48632A  
.text:0068AF8E                 push    520h            ; uBytes  
.text:0068AF93                 push    40h             ; uFlags  
.text:0068AF95                 call    ds:LocalAlloc      ;分配内存  
.text:0068AF9B                 cmp     eax, ebx  
.text:0068AF9D                 jz      short loc\_68AFF8  
.text:0068AF9F                 push    eax        ;保存分配的内存指针  
.text:0068AFA0                 mov     ecx, \[ebp+824h+lpWideCharStr\]    ;将 WCHAR 转换成多字节  
.text:0068AFA3                 mov     edx, eax  
.text:0068AFA5  
.text:0068AFA5 loc\_68AFA5:                             ; CODE XREF: sub\_4861D0+204DE3 j  
.text:0068AFA5                 movzx   eax, word ptr \[ecx\]  
.text:0068AFA8                 mov     \[edx\], al  
.text:0068AFAA                 add     ecx, 2  
.text:0068AFAD                 add     edx, 1  
.text:0068AFB0                 cmp     ax, bx  
.text:0068AFB3                 jnz     short loc\_68AFA5  
.text:0068AFB5                 pop     eax    
.text:0068AFB6                 push    eax  
.text:0068AFB7                 sub     edx, eax       ;计算字符串长度  
.text:0068AFB9                 push    edx             ; cchWideChar  
.text:0068AFBA                 mov     edx, \[ebp+824h+lpWideCharStr\]  
.text:0068AFBD                 push    edx             ; lpWideCharStr  
.text:0068AFBE                 push    0FFFFFFFFh      ; cbMultiByte  
.text:0068AFC0                 push    eax             ; lpMultiByteStr  
.text:0068AFC1                 push    0               ; dwFlags  
.text:0068AFC3                 push    0FDE9h          ; CodePage  
.text:0068AFC8                 call    ds:MultiByteToWideChar    ;转换成 UNICODE 到原来的字符串内存池  
.text:0068AFCE                 jmp     short loc\_68AFF0  
.text:0068AFCE ; END OF FUNCTION CHUNK FOR sub\_4861D0  
.text:0068AFCE ; ---------------------------------------------------------------------------  
.text:0068AFD0                 dd 8 dup(0)  
.text:0068AFF0 ; ---------------------------------------------------------------------------  
.text:0068AFF0 ; START OF FUNCTION CHUNK FOR sub\_4861D0  
.text:0068AFF0  
.text:0068AFF0 loc\_68AFF0:                             ; CODE XREF: sub\_4861D0+204DFE j  
.text:0068AFF0                 pop     eax  
.text:0068AFF1                 push    eax             ; hMem  
.text:0068AFF2                 call    ds:LocalFree     ;释放申请的内存  
.text:0068AFF8  
.text:0068AFF8 loc\_68AFF8:                             ; CODE XREF: sub\_4861D0+204DCD j  
.text:0068AFF8                 lea     ecx, \[ebp+824h+lpWideCharStr\]  恢复原位置代码  
.text:0068AFFB                 jmp     loc\_4862F3    ;返回原跳转处  
.text:0068AFFB ; END OF FUNCTION CHUNK FOR sub\_4861D0  
.text:0068AFFB \_text           ends.text:0068AFF0 ; START OF FUNCTION CHUNK FOR sub\_4861D0  
  
再用神奇 F5 看看，相应的代码从：  
   while ( CStdioFile\_\_ReadString(&v50) )  
      {  
        sub\_476130(&v50);  
        sub\_476220(&v50);  
        v5 = v50;  
        v4 = &v55;  
        do  
        {  
          v6 = \*(\_WORD \*)v5;  
          \*(\_WORD \*)v4 = \*(\_WORD \*)v5;  
          v5 += 2;  
          v4 += 2;  
        }  
        while ( v6 );  
        sub\_484850(&v55, v49, 0, 0);  
      }  
  
变成了:  
      while ( CStdioFile\_\_ReadString(&lpWideCharStr) )  
      {  
        //对应补丁代码  
        v29 = LocalAlloc(0x40u, 0x520u);  
        if ( v29 )  
        {  
          v34 = (int)v29;  
          v31 = lpWideCharStr;  
          v30 = v29;  
          do  
          {  
            v32 = \*v31;  
            \*(\_BYTE \*)v30 = \*v31;  
            ++v31;  
            ++v30;  
          }  
          while ( v32 );  
          MultiByteToWideChar(CP\_UTF8, 0, (LPCSTR)v34, -1, lpWideCharStr, (int)((char \*)v30 - v34));  
          LocalFree((HLOCAL)v34);  
        }  
        sub\_476130();  
        sub\_476220(&lpWideCharStr);  
        v6 = lpWideCharStr;  
        v5 = &v59;  
        do  
        {  
          v4 = \*v6;  
          \*(\_WORD \*)v5 = \*v6;  
          ++v6;  
          v5 += 2;  
        }  
        while ( v4 );  
        sub\_484850(&v59, v53, 0, 0);  
      }  
  
     经测试，在 XP SP3 平台运行效果良好。

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || HWorks32.part1.rar  
 |
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || HWorks32.part2.rar  
 |