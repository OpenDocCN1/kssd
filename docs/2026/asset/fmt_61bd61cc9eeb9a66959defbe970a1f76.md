
可以看到，函数首先将一个数组地址赋给`esi`，然后将`0`压入栈中并且调用`SetArray`这个函数，在调用这个函数之前还把`esi`的地址，也就是这个数组的地址赋给了`eax`，把`ebx`的值赋给`ecx`，这个`ebx`的值就是在上面调用完`AllocateAssign`函数以后得到的分配和赋值的内存地址。因为在调用`AllocateAssign`函数后的返回值`eax`赋值给了`ebx`。

调用完`SetArray`函数以后，会判断返回值是否为`0`，不为`0`那就是调用成功，也就是回调函数设置成功，就会跳转到函数调用成功的退出代码执行。如果为`0`说明并没有设置完成，随后会对`edi`和`esi`进行加`4`的操作并判断`edi`是否小于`0x100`，如果小于就跳上去继续执行`SetArray`函数。由于此时`esi`指向的是`ProcessFuncArray`数组的地址，所以加`4`就是指向数组中下一个元素，`edi`就是为了避免数组溢出而设置，根据值的大小可以猜测这个数组一共`0x40`个大小。

分析到这里就可以得出结论，`PsSetCreateProcessNotifyRoutine`函数设置进程监控的回调函数的办法就是首先申请一块内存来保存要设置的回调函数的地址，然后将这个申请到的内存的地址赋到`ProcessFuncArray`数组中的某个元素。

那么它是怎么判断要赋值到哪个元素呢，就要继续跟进`SetArray`函数

### `SetArray` 函数入口

```
PAGE:00594706 SetArray        proc near               ; CODE XREF: IoRegisterPriorityCallback+5B↑p

PAGE:00594706                                         ; IoUnregisterPriorityCallback+4C↑p ...

PAGE:00594706

PAGE:00594706 var_8           = dword ptr -8

PAGE:00594706 var_4           = dword ptr -4

PAGE:00594706 arg_Zero        = dword ptr  8

PAGE:00594706

PAGE:00594706                 mov     edi, edi

PAGE:00594708                 push    ebp

PAGE:00594709                 mov     ebp, esp

PAGE:0059470B                 push    ecx             ; 此时ecx是分配的地址，eax是数组的地址

PAGE:0059470C                 push    ecx

PAGE:0059470D                 push    ebx

PAGE:0059470E                 push    esi

PAGE:0059470F                 push    edi

PAGE:00594710                 mov     esi, eax        ; 将数组地址赋给esi

PAGE:00594712                 test    ecx, ecx        ; 判断分配地址是否为0

PAGE:00594714                 jz      short loc_594726 

PAGE:00594716                 push    8

PAGE:00594718                 pop     edx

PAGE:00594719                 call    ExAcquireRundownProtectionEx ; 为了让调用者安全访问对象，这里不是重点，过

PAGE:0059471E                 test    al, al

PAGE:00594720                 jz      loc_5947D0      ; 返回为0就代表函数运行失败，会退出函数

PAGE:00594726

PAGE:00594726 loc_594726:                             ; CODE XREF: SetArray+E↑j

PAGE:00594726                 mov     ebx, [esi]      ; 将数组中的内容取出赋给ebx

PAGE:00594728                 mov     eax, ebx        ; 将内容赋给eax

PAGE:0059472A                 jmp     short loc_594749
```

函数将数组中的对应的内容赋给`eax`以后跳转到`loc_594759`执行，接着看`loc_594749`处的代码

```
PAGE:00594749 loc_594749:                             ; CODE XREF: SetArray+24↑j

PAGE:00594749                 xor     eax, [ebp+arg_Zero] ; 将得到的值与0进行异或，与7做比较，小于7则跳转.

PAGE:0059474C                 cmp     eax, 7          ; 所以这两条合起来看的话，就是除了低3位，其他都为0，否则就跳转

PAGE:0059474F                 jbe     short loc_59472C ; 可以得出结论，数组中元素高29位用来判断是否还未使用。

PAGE:0059474F                                         ; 如果为0，说明还未使用
```

在这里程序将得到的数组中对应的元素内容和传入的参数进行异或，然后判断是否小于`7`，如果小于`7`就跳转到对数组进行赋值的代码进行执行。而`0x7`对应的二进制是高`29`位是`0`，低`3`位是`1`，在跟进得到的数组内容要和`0`进行异或可以知道，这里是判断得到的数组中的内容高`29`位是不是`0`。如果高`29`位中有`1`个是`1`，那么异或以后得到的值就会大于`7`。

接着看`loc_59472C`中的代码，也就是对数组进行赋值的代码

```
PAGE:0059472C loc_59472C:                             ; CODE XREF: SetArray+49↓j

PAGE:0059472C                 test    ecx, ecx        ; ecx是申请的地址，判断是否为0，为0则跳转

PAGE:0059472E                 jz      short loc_594737

PAGE:00594730                 mov     eax, ecx    ; 将ecx赋给eax

PAGE:00594732                 or      eax, 7      ; 将低三位置1，这里要注意，在将申请到的内存地址复制到数组之前，会将地址的低3位置1

PAGE:00594735                 jmp     short loc_594739 

PAGE:00594737 ; ---------------------------------------------------------------------------

PAGE:00594737

PAGE:00594737 loc_594737:                             ; CODE XREF: SetArray+28↑j

PAGE:00594737                 xor     eax, eax        ; 如果ecx是0，那就会跳转到这里执行，也就是eax会被变成0

PAGE:00594739

PAGE:00594739 loc_594739:                             ; CODE XREF: SetArray+2F↑j

PAGE:00594739                 mov     edx, eax        ; 将或运算得到的值赋给edx

PAGE:0059473B                 mov     edi, esi        ; 取出数组对应元素地址给edi

PAGE:0059473D                 mov     eax, ebx        ; ebx保存的是数组中的内容，赋给eax

PAGE:0059473F                 lock cmpxchg [edi], edx ; edi是数组地址，而eax是数组元素，所以eax==[edi]

PAGE:0059473F                                         ; 那么此时，edx，也就是计算以后的地址会赋值给[edi]，也就是赋值到数组中

PAGE:00594743                 cmp     eax, ebx        ; 由于上面的赋值操作，这里会成立跳转

PAGE:00594745                 jz      short loc_594751 

PAGE:00594747                 mov     ebx, eax
```

这里的`ecx`的值就是前面用来保存回调函数地址所调用的`AllocateAssign`函数得到的分配的内存地址。因为上面申请到内存以后，将内存地址先赋给`eax`在赋给`ecx`，随后的代码中都没对`ecx`进行改变。

将这个地址赋给`eax`并且与`7`或运算，那就是将低`3`位置`1`，接着就跳转到`loc_594739`处执行。执行的时候，这个`esi`就是要赋值的数组的对应元素的地址，因为在进入`SetArray`函数前，将数组的对应元素的地址赋给了`esi`后就没在改变了。而根据`cmpxchg`的功能可以知道，这里就是将`AllocateAssign`分配到的内存地址和`7`或运算以后的地址复制到数组中，随后跳转到`loc_594751`执行。那么继续看`loc_594751`执行的代码

```
PAGE:00594751 loc_594751:                             ; CODE XREF: SetArray+3F↑j

PAGE:00594751                 mov     edi, ebx        ; ebx中是数组中原来的内容，将他赋值给edi

PAGE:00594753                 and     edi, 0FFFFFFF8h ; 将低3位清0

PAGE:00594756                 cmp     edi, [ebp+arg_Zero] ; 判断是否为0，不为0则跳转退出函数,函数执行失败，返回值为0

PAGE:00594759                 jnz     short loc_5947C4 ; 这里就是判断原来的高29位是不是0，不是的话，函数执行失败

PAGE:0059475B                 test    edi, edi         ;判断edi是否位0

PAGE:0059475D                 jz      short loc_5947C0 ; 为0则跳转，函数执行成功，返回值eax=1

PAGE:0059475F                 mov     esi, large fs:124h ; 接下来的内容和APC相关，这里不讨论

PAGE:00594766                 dec     word ptr [esi+86h]

PAGE:0059476D                 xchg    eax, [ebp+var_4]

PAGE:00594770                 mov     ecx, dword_53C7D8

PAGE:00594776                 and     ecx, 1

PAGE:00594779                 xchg    eax, [ebp+var_8]

PAGE:0059477C                 test    ecx, ecx

PAGE:0059477E                 jz      short loc_594794

PAGE:00594780                 mov     ecx, offset dword_53C7D8

PAGE:00594785                 call    ExfAcquirePushLockExclusive

PAGE:0059478A                 mov     ecx, offset dword_53C7D8

PAGE:0059478F                 call    ExfReleasePushLockExclusive
```

在这里，程序首先将原来数组中保存的内容的低三位清`0`，接着判断得到的数据是否为`0`。也就是判断原来数组中的内容的高`29`位是不是`0`，如果是`0`说明上面的赋值成功了，程序就会跳转到函数执行成功的地方，也就是`loc_5947C0`处继续执行，在`loc_5947C0`，程序会将`eax`赋值为`1`，也就是说返回值是`1`。如果高`29`位不是`0`，那上面的赋值就会失败，程序就会跳转到`loc_5947C4`的地址继续执行，在这个地址中，程序会将`eax`赋值为`0`，也就是说返回值是`0`。

**综上，可以得出以下的结论**

- 程序会申请一块`0xC`大小的内存，内存中低`4`为是`0`，中间`4`为是要设置的回调函数的地址，高`4`为根据是否是`Ex`函数来设置为是`0`还是`1`。

### 原始调用片段

```
PAGE:0059493C loc_59493C:                             ; CODE XREF: PspSetCreateProcessNotifyRoutine+13A↑j

PAGE:0059493C                 mov     esi, offset ProcessFuncArray ; 将进程数组地址赋给esi

PAGE:00594941                 xor     edi, edi        ; 为edi清0

PAGE:00594943

PAGE:00594943 loc_594943:                             ; CODE XREF: PspSetCreateProcessNotifyRoutine+165↓j

PAGE:00594943                 push    0               ; 将0压入栈中

PAGE:00594945                 mov     ecx, ebx        ; 分配到的内存的地址赋值给ecx

PAGE:00594947                 mov     eax, esi        ; 数组地址赋值给eax

PAGE:00594949                 call    SetArray

PAGE:0059494E                 test    al, al

PAGE:00594950                 jnz     short loc_594972 ; 函数返回值不为0，说明设置成功

PAGE:00594952                 add     edi, 4          ; 将edi加4

PAGE:00594955                 add     esi, 4          ; 数组的地址加4，也就是获得下一个元素的地址

PAGE:00594958                 cmp     edi, 100h       ; 判断是否小于0x100，小于的话。使用新地址来调用上面的函数继续设置

PAGE:0059495E                 jb      short loc_594943 ; 这里可以得出结论，这个数组一共0x100 % 4 = 0x40

PAGE:0059495E                                         ; 也就是说，最多可以设置64个进程创建的回调函数

PAGE:00594960                 push    ebx             ; Buffer

PAGE:00594961                 call    FreeAllocate

PAGE:00594966                 mov     eax, STATUS_INVALID_PARAMETER ; 如果执行到这里说明赋值失败

PAGE:00594966                                         ; 接下来就释放内存，返回值设为失败

PAGE:0059496B

PAGE:0059496B loc_59496B:                             ; CODE XREF: PspSetCreateProcessNotifyRoutine+A8↑j

PAGE:0059496B                                         ; PspSetCreateProcessNotifyRoutine+10A↑j ...

PAGE:0059496B                 pop     edi

PAGE:0059496C                 pop     esi

PAGE:0059496D                 pop     ebx

PAGE:0059496E                 pop     ebp

PAGE:0059496F                 retn    0Ch
```
