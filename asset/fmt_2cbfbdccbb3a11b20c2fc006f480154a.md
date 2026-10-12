
这里说明一点，代码在每次处理一个目录项或者资源数据的时候，都将它们的名称或 ID 等信息显示出来。如果例子中的代码被移植到了其他地方用来寻找资源的话，这些显示信息的语句就可以全部去掉了，因为这时程序的最终目的就是最后两句获取资源 RVA 和大小的指令。

### 得到 PE 文件的重定位表

重定位所使用的一个`_ProcessPeFile`来处理，我们来看看这个函数是怎么样实现的：

```asm
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

_ProcessPeFile  proc  _lpFile,_lpPeHead,_dwSize                 ;传入三个参数，文件，PE 文件头，大小

    local  @szBuffer[1024]:byte,@szSectionName[16]:byte

    pushad

    mov  esi,_lpPeHead

    assume  esi:ptr IMAGE_NT_HEADERS                  ;定义 PE 文件头

;********************************************************************

; 根据 IMAGE_DIRECTORY_ENTRY_BASERELOC 目录表找到重定位表位置

;********************************************************************

    mov  eax,[esi].OptionalHeader.DataDirectory[8*5].VirtualAddress   ;根据 PE 文件头定位到重定位表

    .if  ! eax

      invoke  MessageBox,hWinMain,addr szErrNoReloc,NULL,MB_OK

      jmp  _Ret

    .endif

    push  eax

    invoke  _RVAToOffset,_lpFile,eax                        ;将重定位表的 RVA 转换为 File Offset

    add  eax,_lpFile

    mov  esi,eax

    pop  eax

    invoke  _GetRVASection,_lpFile,eax                   ;得到重定位表所处的区块名

    invoke  wsprintf,addr @szBuffer,addr szMsg,addr szFileName,eax

    invoke  SetWindowText,hWinEdit,addr @szBuffer

    assume  esi:ptr IMAGE_BASE_RELOCATION

;********************************************************************

; 循环处理每个重定位块

;********************************************************************

    .while  [esi].VirtualAddress                        ;以重定位表在内存中的起始 RVA 为循环条件

      cld                              ;设置方向标志 DF

      lodsd      ;eax = [esi].VirtualAddress   重定位内存页的起始 RVA

      mov  ebx,eax

      lodsd      ;eax = [esi].SizeOfBlock   重定位块的长度

      sub  eax,sizeof IMAGE_BASE_RELOCATION

      shr  eax,1        ;重定位项的数量 n 就等于（SizeOfBlock-sizeof IMAGE_BASE_RELOCATION)/2

      push  eax    ;eax = 重定位项数量

      invoke  wsprintf,addr @szBuffer,addr szMsgRelocBlk,ebx,eax

      invoke  _AppendInfo,addr @szBuffer

      pop  ecx

      xor  edi,edi

      .repeat

        push  ecx

        lodsw

        mov  cx,ax

        and  cx,0f000h         ;取重定位项的高四位得到重定位项的类型

;********************************************************************

; 仅处理 IMAGE_REL_BASED_HIGHLOW 类型的重定位项

;********************************************************************

        .if  cx ==  03000h

          and  ax,0fffh  ;取重定位项的低十二位得到重定位项的地址

          movzx  eax,ax

          add  eax,ebx   ;低十二位加上前面得到的重定位内存页的 RVA（虚拟地址）

        .else

          mov  eax,-1

        .endif

        invoke  wsprintf,addr @szBuffer,addr szMsgReloc,eax    ;格式化输出

        inc  edi         ;EDI 自增 1

        .if  edi ==  4  ;每显示 4 个项目换行

          invoke  lstrcat,addr @szBuffer,addr szCrLf

          xor  edi,edi

        .endif

        invoke  _AppendInfo,addr @szBuffer

        pop  ecx

      .untilcxz

      .if  edi

        invoke  _AppendInfo,addr szCrLf

      .endif

    .endw

_Ret:

    assume  esi:nothing

    popad

    ret

_ProcessPeFile  endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
```

参考资料《Win32 汇编语言程序设计》老罗的第二版！

上面的注释已经讲的很清楚了，我就不重复了，如果大家真的把前面的几篇文章弄懂了，我想看上面的代码并不困难，就是将前面几章的内容进行编程实现，然后按规定的格式显示出来了，PE 文件的复习就基本上就完了，在接下来的两天里，我会讲解几个 PE 文件的编程的实际应用的例子与代码，希望对大家理解 PE 文件有所帮助！`![](img/6821a6175013046370981b6035d43ab4.jpg "使眼色")`

有点累了，先睡了，鼻子很不舒服，可能是感冒了，不断流鼻涕`![](img/b0e6af7069663aededb11e4e29a3e6d1.jpg "碌眼")`，不过还是写完了！`![](img/cda69129882b40377a87144c782680e1.jpg "微笑")`

我倒，本来我上面的图片很清楚的，不知道为什么传上去了，这么模糊！！`![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")`算了，还是在下面帖出这两个函数的源代码吧！！

```asm
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

; 将 RVA 转换成实际的数据位置

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

_RVAToOffset  proc  _lpFileHead,_dwRVA                 ;这个函数带两个参数，一个是映射头，还有一个就是 RVA 的值

    local  @dwReturn

    pushad

    mov  esi,_lpFileHead

    assume  esi:ptr IMAGE_DOS_HEADER

    add  esi,[esi].e_lfanew

    assume  esi:ptr IMAGE_NT_HEADERS

    mov  edi,_dwRVA

    mov  edx,esi

    add  edx,sizeof IMAGE_NT_HEADERS           ;PE 文件头加上 PE 文件头的大小，得到区块的首地址

    assume  edx:ptr IMAGE_SECTION_HEADER        ;定义 EDX 为区块

    movzx  ecx,[esi].FileHeader.NumberOfSections    ;以 PE 文件的区块个数作为循环条件，进行循环

;********************************************************************

; 扫描每个节区并判断 RVA 是否位于这个节区内

;********************************************************************

    .repeat

      mov  eax,[edx].VirtualAddress     ;VirtualAddress 是区块的 RVA 地址，即表示这个区块的开头位置

      add  eax,[edx].SizeOfRawData     ;SizeofRawData 是在文件对齐后的尺寸，即表示这个区块的结尾位置

      .if  (edi >= [edx].VirtualAddress) && (edi < eax)

        mov  eax,[edx].VirtualAddress ;VirtualAddress 是区块的 RVA 地址，即表示这个区块的开头位置

        sub  edi,eax       ;EDI 中存放的是这个 RVA 在内存中相对于起始 RVA 的偏移量

        mov  eax,[edx].PointerToRawData ;PointerToRawData 表示文件中的偏移起始地址

        add  eax,edi       ;EAX 存放的是 RVA 转换成的 File Offset 的值

        jmp  @F                   ;结束循环

      .endif

      add  edx,sizeof IMAGE_SECTION_HEADER  ;EDX 指向下一个区块

    .untilcxz

    assume  edx:nothing

    assume  esi:nothing

    mov  eax,-1                     ;如果不在区块之类，则返回-1

@@:

    mov  @dwReturn,eax             ;将返回值传到@dwReturn 中保存

    popad

    mov  eax,@dwReturn

    ret

_RVAToOffset  endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

; 查找 RVA 所在的节区

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

_GetRVASection  proc  _lpFileHead,_dwRVA        ;这个函数带两个参数，一个是映射文件头，一个是 RVA 的值

    local  @dwReturn

    pushad

    mov  esi,_lpFileHead

    assume  esi:ptr IMAGE_DOS_HEADER

    add  esi,[esi].e_lfanew

    assume  esi:ptr IMAGE_NT_HEADERS

    mov  edi,_dwRVA

    mov  edx,esi

    add  edx,sizeof IMAGE_NT_HEADERS           ;PE 文件头加上 PE 文件头的大小，得到区块的首地址

    assume  edx:ptr IMAGE_SECTION_HEADER        ;定义 EDX 为区块

    movzx  ecx,[esi].FileHeader.NumberOfSections    ;以 PE 文件的区块个数作为循环条件，进行循环

;********************************************************************

; 扫描每个节区并判断 RVA 是否位于这个节区内

;********************************************************************

    .repeat

      mov  eax,[edx].VirtualAddress     ;VirtualAddress 是区块的 RVA 地址，即表示这个区块的开头位置

      add  eax,[edx].SizeOfRawData     ;SizeofRawData 是在文件对齐后的尺寸，即表示这个区块的结尾位置

      .if  (edi >= [edx].VirtualAddress) && (edi < eax)

        mov  eax,edx       ;EAX 存储的区块的名字

        jmp  @F

      .endif

      add  edx,sizeof IMAGE_SECTION_HEADER  ;指向下一个区块

    .untilcxz

    assume  edx:nothing

    assume  esi:nothing

    mov  eax,offset szNotFound

@@:

    mov  @dwReturn,eax             ;将区块的名字传给@dwReturn

    popad

    mov  eax,@dwReturn

    ret
```
