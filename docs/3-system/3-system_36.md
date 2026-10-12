
# PE 文件中, `RVA` 和文件偏移的转换(来分享下)

> 原文：[`www.pediy.com/kssd/pediy11/104759.html`](https://www.pediy.com/kssd/pediy11/104759.html)

最近在学习 `PE` 文件, 被 `RVA` 和文件偏移搞的非常郁闷, 非常之纠结… 不过还好… 总算整明白了, 现整理下… 理下思路…. 就从罗云彬的教材中的例子说起吧…

```asm
.const  

szNotFound   db     '无法查找', 0  

     .code  

;>>>>>>>>>>>>>>>>>  

;将 RVA 转换为文件偏移  

;>>>>>>>>>>>>>>>>>  

_RVAToOffset   proc _lpFileHead, _dwRVA  

               Local @dwReturn  

               Pushad  

               Mov  esi, _lpFileHead  

               Assume esi : ptr IMAGE_DOS_HEADER ; esi 指向 DOS MZ 文件头  

               Add  esi, [esi].e_lfanew ;esi 指向 IMAGE_NT_HEADERS 结构  

;****************************************  

;得到 PE 文件头的位置, 这个位置加上文件头长度就是节表的位置  

;****************************************  

               Assume  esi: ptr IMAGE_NT_HEADERS  

               Mov  edi, _dwRVA ; edi 中存在数据在内存中的 RVA  

               Mov  edx, esi  

               Add  edx, sizeof IMAGE_NT_HEADERS  

 ;ebx 指向 IMAGE_SECTION_HEADER 结构  

              Assume  edx: ptr IMAGE_SECTION_HEADER  

               Mozx  ecx, [esi].FileHeader.NumberOfSections ;ecx 存放节的数量  

;**********************************************************************  

;扫描每个节并判断 RVA 是否存在于这个节内  

;**********************************************************************  

               .repeat  

                      Mov  eax, [edx].VirtualAddress  

;每个节的数据的起始位置相对于节的起始位置是不变的, 不管节是在文件中还是在文件中  

                      Add  [ebx].SizeOfRawData ; 节的大小  

                      .if    (edi >= [ebx].VirtualAddress) && (edi < eax) ;数据在该节内  

                              Mov  eax, [ebx].VirtualAddress  

                              Sub   edi, eax ;数据相对于节的起始地址的偏移地址 RVA’  

                              Mov  eax, [ebx].PointerToRawData  

                              Add   eax, edi ;数据在文件中的偏移地址  

                              Jmp   @F  

                      .endif  

                      Add  edx, sizeof IMAGE_SECTION_HEADER  

                  .untilcxz  

                  Assume  edx:nothing  

                  Assume  esi: nothing  

                  Mov     eax, -1  

@@:  

                  Mov     @dwReturn, eax  

                  Popad  

                  Mov  eax, @dwReturn  

                  Ret  

_RVATOffset     endp  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
```
