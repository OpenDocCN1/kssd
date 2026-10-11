
# 汇编代码分析

## 1. 初始汇编代码
```asm
mov  eax,004cd25c  
call  0046fca0  
sub  esp,48h  
push  ebx  
push  esi  
xor  ebx,ebx  
mov  [ebp-4],ebx ;[ebp-4] 是 seh 附加数据的地址，原来值为-1,现在要把它置零了  
mov  @dwVar1,ebx  
mov  ecx,_arg1  
mov  eax,[ecx+84] ；可知_arg1 是一个指针  
lea  edx,@dwVar1  
push  edx  
push  004e7460  
mov  ecx,[eax]  
push  eax  
mov  byte ptr[ebp-4],1 ;[ebp-4] 是 seh 附加数据的地址  
call  [ecx+1c]    ;表明_arg1 是一个函数指针的指针的指针  
test  eax,eax  
jnz  lable1  
mov  eax,@dwVar1  
lea  edx,@dwVar2  
push  edx  
push  004e8940    ；ascii "ewh.db"  
mov  ecx,[eax]  
push  eax  
call  [ecx+14]  
test  eax,eax  
je  label2  
  lable1:  
    mov  eax,_arg4    ;可知_arg4 是一个 int*  
    mov  dword ptr[eax],2  
    jmp  lable3  
  lable2:  
    mov  eax,_arg3  
    push  edi  
    mov  edx,@dwVar1  
    push  1  
    mov  ecx,[eax-8]  
    pop  esi  
    mov  edi,[edx]  
    push  esi  
    push  eax  
    push  ecx  
    push  ebx  
    push  dword ptr _arg2  
    push  edx  
    call  [edi+1c]  
    test  eax,eax  
    pop  edi  
    je  lable4   
    cmp  _arg6,esi  
    je  lable5  
    mov  eax,_arg4  
    mov  [eax],ebx ;可知_arg4 是个指针  
    jmp  lable3  
  lable5:  
    lea  ecx,_arg6  
    call  CString::CString  
    lea  ecx,_arg2  
    mov  byte  ptr [ebp-4],2  
    call  CString::CString  
    mov  esi,BasicCtrDll.BasicLoadStr  
    lea  eax,_arg6  
    push  281  
    push  eax  
    mov  byte  prt[ebp-4],3  
    call  esi  
    lea  eax,_arg2  
    push  28d  
    push  eax  
    call  esi  
    add  esp,10  
    lea  ecx,[ebp-54]  ;ebp-54 指向一个结构  
    call  CWNd::CWnd  
    push  114  
    lea  ecx,[ebp-54]  
    push  dword  ptr _arg2  
    mov  byte  prt[ebp-4],4  
    push  dword  ptr _arg6  
    call  CWnd::MessageBoxA  
    cmp  eax,6  
    mov  eax,_arg4  
    jnz  Lable6  
    mov  dword  prt[eax],2  
    jmp  lable7  
  lable6:  
    mov  [eax],ebx  
  lable7:  
    lea  ecx,[ebp-54]  
    mov  byte  prt[ebp-4],3  
    call  CWnd::~Cwnd  
    lea  ecx,_arg2  
    mov  byte  prt[ebp-4],2  
    call  CString::~CString  
    lea  ecx,_arg6  
    mov  byte  prt[ebp-4],1  
    call  CString::~Cstring  
    jmp  lable3  
  lable4:  
    mov  eax,_arg4  
    mov  [eax],esi  
  lable3:  
    mov  eax,@dwVar1  
    mov  [ebp-4],bl  
    cmp  eax,ebx  
    je  lable8  
    mov  ecx,[eax]  
    push  eax  
    call  [ecx+8]  
  lable8:  
    or  dwword  prt[ebp-4],0ffffffffh  
    lea  ecx,_arg3  
    call  CString::~CString  
    mov  ecx,[ebp-c]  ; pre seh handler 指针  
    pop  esi  
    pop  ebx  
    mov  fs:[0],ecx  
    leave  
    retn  14  
_LastSetp  enp  
```

今天就到这里，明天继续

`2006-6-15 13:20`

欢迎访问我的博客：`http://dreamcometotrue.blog.ccidnet.com`

## 2. 识别函数中的参数和局部变量

经过上面的分析我们已经知道在 `qq00405b55` 函数中参数和局部变量都是用 `ebp` 寄存器来表示的,现在我们来找出函数中的参数，和局部变量

首先：我们把函数中使用 `ebp` 寄存器寻找的变量找出来，删除重复的并按从低地址到高地址排序，得到如下结果：

`ebp-54h`

`ebp-14h`

`ebp-10h`

`ebp-ch`

`ebp-4h`

`ebp+8h`

`ebp+ch`

`ebp+10h`

`ebp+18h`

我们知道堆栈参数是：从 `ebp+8` 开始向高地址方向的，局部变量是从 `ebp-d` 开始向低地址方向的

现在给他们分组：

堆栈参数有：`ebp+8h`, `ebp+ch`, `ebp+10h`, `ebp+18h`, 堆栈参数名我们规定从低地址开始编号 `_arg1`, `_arg2`, `_arg3`, `....`, 所以就有 ：

`ebp+8h` = `_arg1`

`ebp+ch` = `_arg2`

`ebp+10h` = `_arg3`

`ebp+18h` = `_arg4` 少了一个 怎么没有 `ebp+14h` 呢， `_arg3` 占了 `8` 个字节的地盘？？？先不管它，呆会分析参数类型的时候在看

局部变量有：`ebp-10h`, `ebp-14h`, `ebp-54h`, 局部变量我们规定从高地址开始编号 `@var1`, `@var2`, `@var3`, `.....` 所以有:

`ebp-10h` = `@var1`  因为局部变量是从 `ebp-d` 开始的所以可知这个局部变量的大小为 `(10h-d)+1`  `4` `byte`

`ebp-14h` = `@var2` 同理：这个紧挨着 `@var1`  可知他也为 `(14h-11h)+1`  `4` `byte`

`ebp-54h` = `@var3` 这是最后一个变量，它的大小为：`(54h-15h)+1` = `40h`  `byte`

还有几个孤儿，没有组的，他们是 `ebp-ch`, 和 `ebp-4h`，日至 `1` 中已经知道他们分别是 `seh` 指针和 `seh` 的附加数据

第二步：我们将参数名、局部变量名替换到反汇编代码中去，还有跳转地址我们也用相应的标号替换掉就得到了

```asm
  mov     eax, 004CD25C  
  call    0046FCA0  
  sub     esp, 48  
  push    ebx  
  push    esi  
  xor     ebx, ebx  
  mov     [ebp-4], ebx  
  mov     @var1, ebx  
  mov     eax, [ecx+84]  
  lea     edx, @var1  
  push    edx  
  push    004E7460  
  mov     ecx, [eax]  
  push    eax  
  mov     byte ptr [ebp-4], 1  
  call    [ecx+1C]  
  test    eax, eax  
  jnz     lable1    ;short 00415B9F  
  mov     eax, @var1  
  lea     edx, @var2  
  push    edx  
  push    004E8940  
  mov     ecx, [eax]  
  push    eax  
  call    [ecx+14]  
  test    eax, eax  
  je      lable2    ;short 00415BAD  
lable1:  
  mov     eax, _arg3  
  mov     dword ptr [eax], 2  
  jmp     lable3    ;00415C72  
lable2:  
  mov     eax, _arg2  
  push    edi  
  mov     edx, @var1  
  push    1  
  mov     ecx, [eax-8]  
  pop     esi  
  mov     edi, [edx]  
  push    esi  
  push    eax  
  push    ecx  
  push    ebx  
  push    dword ptr _arg1  
  push    edx  
  call    [edi+1C]  
  test    eax, eax  
  pop     edi  
  je      lable4    ;00415C6D  
  cmp     _arg4, esi  
  je      lable5    ;short 00415BDF  
  mov     eax, _arg3  
  mov     [eax], ebx  
  jmp     lable3    ;00415C72  
lable5:  
  lea     ecx, _arg4  
  call    <jmp.&MFC42.#540_CString::CStrin>  
  lea     ecx, _arg1  
  mov     byte ptr [ebp-4], 2  
  call    <jmp.&MFC42.#540_CString::CStrin>  
  mov     esi, [<&BasicCtrlDll.BasicLoadSt>;  BasicCtr.BasicLoadStr  
  lea     eax, _arg4  
  push    281  
  push    eax  
  mov     byte ptr [ebp-4], 3  
  call    esi                   ;  <&BasicCtrlDll.BasicLoadStr>  
  lea     eax, _arg1  
  push    28D  
  push    eax  
  call    esi  
  add     esp, 10  
  lea     ecx, @var3  
  call    <jmp.&MFC42.#567_CWnd::CWnd>  
  push    114  
  lea     ecx, @var3  
  push    dword ptr _arg1  
  mov     byte ptr [ebp-4], 4  
  push    dword ptr _arg4  
  call    <jmp.&MFC42.#4224_CWnd::MessageB>  
  cmp     eax, 6  
  mov     eax, _arg3  
  jnz     label6        ;short 00415C45  
  mov     dword ptr [eax], 2          ;  [eax]可能保存的是某个标志，2 代表本地登陆失败，到服务器上去验证去  
  jmp     lable7        ;short 00415C47  
lable6:  
  mov     [eax], ebx  
lable7:  
  lea     ecx, @var3  
  mov     byte ptr [ebp-4], 3          ;  [12fcac]  
  call    <jmp.&MFC42.#818_CWnd::~CWnd>  
  lea     ecx, _arg1  
  mov     byte ptr [ebp-4], 2  
  call    <jmp.&MFC42.#800_CString::~CStri>  
  lea     ecx, _arg4  
  mov     byte ptr [ebp-4], 1  
  call    <jmp.&MFC42.#800_CString::~CStri>  
  jmp     lable3        ;short 00415C72  
lable4:  
  mov     eax, _arg3  
  mov     [eax], esi  
lable3:  
  mov     eax, _arg3  
  mov     [ebp-4], bl  
  cmp     eax, ebx  
  je      lable8        ;short 00415C82  
  mov     ecx, [eax]  
  push    eax  
  call    [ecx+8]  
lable8:  
  or      dword ptr [ebp-4], FFFFFFFF  
  lea     ecx, _arg2  
  call    <jmp.&MFC42.#800_CString::~CStri>  
  mov     ecx, [ebp-C]           ;  seh 指针  
  pop     esi  
  pop     ebx  
  mov     fs:[0], ecx  
  leave  
  retn    14                   ;  父函数传递了 5 个参数  
```

靠，终于整完了

## 3. 下一步：去掉编译器自己添加的或与函数功能无关的，异常处理东东

删掉

```asm
mov     eax, 004CD25C  
call    0046FCA0  
注册异常处理函数，打开堆栈页面  
sub     esp, 48    ；分配局部变量空间
```

删掉

```asm
mov     ecx, [ebp-C]             ;  seh 指针
```

删掉

```asm
mov     fs:[0], ecx  
leave  
retn    14  
将注册的异常处理函数删除掉，堆栈平衡
```

剩下的就是裸函数了

```asm
push    ebx  
push    esi  
xor     ebx, ebx  
mov     [ebp-4], ebx  
mov     @var1, ebx  
mov     eax, [ecx+84]  
lea     edx, @var1  
push    edx  
push    004E7460  
mov     ecx, [eax]  
push    eax  
mov     byte ptr [ebp-4], 1  
call    [ecx+1C]  
test    eax, eax  
jnz     lable1  
mov     eax, @var1  
lea     edx, @var2  
push    edx  
push    004E8940  
mov     ecx, [eax]  
push    eax  
call    [ecx+14]  
```
